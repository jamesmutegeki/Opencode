## 2026-11-08 — Fix agent halting, execution stalls, and tool timeouts

Addressed repeated hangs where OpenCode and Claude Code would announce actions in text (e.g., "Now let me check the sizes...") and halt without executing tools, as well as command timeouts (120000 ms):

1. **Anti-Stall & Continuous Execution rules added to `AGENTS.md`**:
   - Forbids plain-text intent statements ("Let me do X", "I will now check Y") without emitting the tool call in the exact same turn. When agents emit text without tool calls, runtime harnesses interpret `finish_reason: "stop"` as yielding control to the user.
   - Mandates continuous autonomous execution for diagnostic, inspection, and repair workflows until completion before returning to prompt.
   - Directs agents to specify higher `timeout` parameters for long-running commands (builds, tests, indexing) instead of relying on the default 120,000 ms tool cutoff.
   - WSL cross-filesystem efficiency: instructs agents never to run recursive tools (`du`, `find`) across `/mnt/c` or `/mnt/d` via the slow WSL 9P filesystem bridge; use native `powershell.exe` interop instead.
   - Clarified Section 2 so agents do not misinterpret "pause and ask" as a reason to stall on read-only exploration and diagnostics.

2. **Fixed Windows OpenCode PATH wrapper conflict**:
   - `C:\Users\james\.opencode\src\tools\opencode.ps1` and `opencode.bat` were shadowing the real npm executable in User PATH and attempting to invoke a non-existent binary (`.../opencode-ai/bin/opencode.exe`) while force-injecting `deepseek-coder` overrides.
   - Rewrote both wrappers to transparently proxy to the official OpenCode 2.x binary at `C:\Users\james\AppData\Roaming\npm\node_modules\@opencode\cli\bin\opencode.exe`.

3. **Mirrored policy to Claude Code in WSL**:
   - Appended identical Anti-Stall, Continuous Execution, and Cross-Filesystem rules to `~/.claude/CLAUDE.md`.

Left deliberately unchanged:
- The provider model in `~/.claude/settings.json` (`nemotron-3-ultra-550b-a55b`) is documented as a key latency contributor (~13.8 tok/s, 2m+ generation times); user instructed to switch models for high-throughput coding if needed.

Verified:
- `opencode --version` runs cleanly and outputs `opencode v2.2.1`.
- `opencode debug config` validates without errors.

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
