## 2026-10-29 — Fix opencode startup: v2 config schema, plugin loading, unauthenticated MCP

opencode 2.2.1 refused to run. Three independent causes, all fixed:

1. **Provider block was invalid for the v2 schema.** `provider.opencode` used the v1
   shape (`npm` + `options` + `models` as plain strings). opencode logged
   `$.provider.opencode kind=invalid` and skipped the whole block, so no models were
   ever available. Rewritten as `providers.fcc` with `package: "@ai-sdk/anthropic"`,
   `settings.baseURL/apiKey`, and `models` as objects carrying `modelID`.
   `npm: "@ai-sdk/openai-compatible"` was also wrong on its own: it calls
   `/v1/chat/completions`, which the FCC proxy does not serve (404). `@ai-sdk/anthropic`
   talks to `/v1/messages`, which does work.

2. **Plugin activation abort.** `plugins[]` listed 13 v1-API packages, and
   `~/node_modules` held 14 more; opencode 2.x auto-loads every `opencode-*` package
   from the project's `node_modules`. All export the old plugin signature, so each
   raised `Plugin must export a default definition with an id and an effect or setup
   function`, and opencode failed the whole location with
   `ServiceUnavailableError: Location plugins did not finish activating`.
   `plugins` is now `[]` and the v1 packages were moved to
   `~/archive/node_modules.opencode-v1/`. Re-install only v2-compatible plugins.

3. **Unauthenticated MCP server.** `supabase` is set to `"enabled": false`; its OAuth
   flow 404s on every start and stalls startup.

Also fixed in this change:

- `default_agent` was `build-orchestrator`, which does not exist; set to `build`.
- Added `"model": "fcc/sonnet-4"` so a default model exists.
- `~/.cache/opencode/npm` (77 cached plugin packages) and
  `~/.cache/opencode/packages` (53) were archived, not deleted, to
  `~/.cache/opencode/*.disabled-20261029`.

Restored / superseded files preserved under `archive/`:

- `archive/opencode.json.pre-2026-10-29.bak` — the broken v1 config.
- `archive/opencode.json.2026-10-27.bak` — previous backup.

Security: `.env` was tracked in git and contains real-looking
`ANTIGRAVITY_API_KEY` / `IEEE_XPLORE_API_KEY` / `SEMANTIC_SCHOLAR_API_KEY` values.
It is now untracked and listed in `.gitignore`. **Those keys are still in the history
of this repo and should be rotated.** No history rewrite was attempted.

Left deliberately unchanged:

- The 70+ skills and agent definitions in `opencode.json` — they are `agent` (v1 key),
  which v2 still accepts as deprecated without error.
- `~/opencode.jsonc` (home-level project config) was updated separately in the home
  checkout: removed the 10 v1 plugins and converted the ollama `provider` block to
  `providers`.
- `~/opencode.json`, the broken archive referenced in AGENTS.md, was moved to
  `~/archive/config/opencode.json.archive`; it must never sit in a directory opencode
  is started from.

Verified: `opencode run` answers and executes tools (~10s) from a project directory.
Note that running opencode **in `$HOME` itself** is slow (12 MCP servers boot and the
git snapshot walks the entire home tree); use a project directory.
