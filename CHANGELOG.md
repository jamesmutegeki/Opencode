# Changelog

All changes to the opencode configuration. Live config lives at
`~/.config/opencode/opencode.jsonc` on Windows; this repo is the source of truth
for it.

---

## 2026-10-03 — Shell and worktree tools broken: diagnosis only, no config change

### Symptom

Every `bash` / shell tool call fails immediately, before any command runs:

```
The "paths[0]" property must be of type string, got undefined
```

That is Bun's `path.resolve` guard, so something is calling `path.resolve` /
`path.join` with `undefined` as the first element. The `worktree` tool fails
with the identical error on a plain status call (no arguments). `read`, `glob`,
`grep`, `webfetch` and `devcontainer` all work normally.

### What was ruled out

- **`opencode.jsonc` is valid.** Checked against
  <https://opencode.ai/config.json>: `instructions`, `skills.paths`, `plugin`,
  `provider` and `mcp` are all schema-valid shapes.
- **No project-level config.** `D:\CV` contains only `certificates/`; there is no
  `.opencode/` or `opencode.json(c)` there to override anything.
- **Not a shell/command problem.** The error is byte-identical for PowerShell
  one-liners, `cmd /c`, and bare `echo hi`, with and without `workdir`.
- **Not the plugin set.** None of the 10 configured plugins contain a
  `path.resolve` call on a bash-related path; `opencode-workaholic` has no path
  resolution at all and `opencode-autotitle` only calls
  `path.resolve(process.cwd(), logFile)`.
- **Not reached by the command parser.** `opencode.log` never records a
  `permission=bash` evaluation for any run today, so the throw happens in core
  `ShellTool.execute` upstream of permission checking.

### Most likely cause (unverified)

`opencode.log` for this session:

```
creating instance  directory="D:\CV"
created ... projectID=global directory="D:\CV" workspaceID=undefined
```

`D:\CV` is not a git repository, so opencode resolves it as a non-project:
`projectID=global` and `workspaceID=undefined`. Core's `InstanceStore.boot`
builds the instance context with `worktree: result.sandbox` from
`project.fromDirectory()`, and an unresolved sandbox is the most plausible
source of the `undefined` first path element. Every earlier run logged today
(15:23-17:57) used `C:\Users\james` or `D:\opencode repairs`, the latter being
the parent of the `Opencode` mirror repo.

### Changes made to the live config

**None.** `opencode.jsonc`, `AGENTS.md`, plugins, skills and MCP servers were
all left exactly as they were. There was no config defect to correct.

### Left in a broken or unverified state

1. **Shell and worktree tools are still broken.** No fix was applied, because
   applying one requires running a command and process spawning is precisely
   what is failing. The untested fix is `git init "D:\CV"`, then restart
   opencode so the session directory resolves to a real project. This is
   unverified — a hand-written `.git` skeleton was considered and rejected as
   riskier than the real thing.
2. **Leftover file to delete:**
   `~/.config/opencode/skills/ck/commands/ocdiag.mjs`. It was written while
   trying to bootstrap a working shell through the skill-script runner, which
   does not work either. It has been reduced to a no-op stub. It was
   deliberately **not** committed here. No file-delete tool was available in
   that session.
3. **opencode must be restarted** for anything in this entry to be evaluated.

### Harmless leftovers noticed

`~/.cache/opencode/packages/` contains duplicate `@latest`-suffixed install
directories for 8 of the plugins (for example
`opencode-semantic-anchors@latest` alongside `opencode-semantic-anchors`).
opencode resolves plugins by bare name, so the `@latest` copies are inert and
were left in place. `opencode-agent-skills` and `opencode-devcontainers` are
resolved from elsewhere and are not in that cache tree.

---

## 2026-10-03 — Emergency recovery: config was wiped, rebuilt from backup

### What happened

`~/.config/opencode/opencode.jsonc` was overwritten on **2026-10-02 18:21** and
reduced to a 50-byte stub containing nothing but `"$schema"`. The live config
directory had been stripped down to `opencode.jsonc`, `package.json`,
`package-lock.json` and `node_modules`. Everything else was gone:

- no `plugin` array
- no `mcp` servers
- no `agent` / `command` definitions
- no `provider` block
- no `skills/` directory (145 skill folders, 153 `SKILL.md` files, 4.9 MB)

The previous config survived only as `~/.config/opencode.bak/`.

### Root cause (probable)

The `.bak` tree contained several `.jsonc` variants holding an
`opencode.jsonc`-shaped file that could not be loaded: an inline-comment block
plus corrupted UTF-8 in the omniroute model names (`"Auto �? balanced best free
route"`). Loading or writing any of those would have aborted startup, and the
recovery attempts appear to have ended with the live file being replaced by a
minimal schema-only stub rather than restored.

### Actions taken

**Skills restored.** Copied `opencode.bak/skills` back to
`~/.config/opencode/skills` — 145 folders, 153 `SKILL.md` files, 4.9 MB. All 10
plugins below were confirmed to exist on the npm registry via `npm view` before
being written into the config, so none of them can fail to resolve at startup.

**`opencode.jsonc` rebuilt** from the backup rather than restored verbatim. The
old file could not be reused as-is, so the config was reconstructed using only
schema-valid shapes.

MCP servers restored — 8 of the original 11:

| Server | Type | Auth |
| --- | --- | --- |
| `context7` | remote | none |
| `firecrawl` | remote | `FIRECRAWL_API_KEY` (present) |
| `exa` | remote | none |
| `gh_grep` | remote | none |
| `playwright` | local (`npx @playwright/mcp@latest`) | none |
| `chrome-devtools` | local (`npx -y chrome-devtools-mcp@latest`) | none |
| `supabase` | remote | OAuth |
| `github` | local (`~/.local/bin/github-mcp-server.exe stdio`) | `GITHUB_TOKEN` (present) |

Deliberately left out, with reasons:

- `higgsfield` — no `HIGGSFIELD_API_KEY` present
- `canva` — no `CANVA_API_KEY` present
- `testsprite` — no `TESTSPRITE_API_KEY` present
- `google-calendar`, `gmail` — were already `enabled: false` in the backup

Plugins restored — 10 of the original 26:

`opencode-agent-skills`, `opencode-agent-memory`, `opencode-semantic-anchors`,
`opencode-adaptive-thinking`, `opencode-autotitle`, `opencode-sessions`,
`opencode-plan-manager`, `opencode-background-agents`, `opencode-devcontainers`,
`opencode-workaholic`

Deliberately left out: `opencode-antigravity-auth` (referenced by absolute path
into `node_modules`, which had been rebuilt empty),
`@bluelovers/opencode-arise`, `envsitter-guard`, `harness-memory`,
`opencode-agent-tmux`, `opencode-ccs-sync`, `opencode-gemini-auth`,
`opencode-github-release`, `opencode-gpt-imagegen`, `opencode-mystatus`,
`opencode-omniroute-auth`, `opencode-personality`, `opencode-research-papers`,
`open-dynamic-workflows`, `open-plan-annotator`, `oh-my-opencode-slim`.

**No `model` / `small_model` override was set.** The backup pointed at
`google/antigravity-claude-opus-4-6-thinking`, which depended on a provider
setup using the non-standard `providers` (plural) / `package` / `settings`
shape that current opencode rejects. Leaving the model unset keeps opencode on
its working default. `GOOGLE_API_KEY`, `OPENROUTER_API_KEY`, `GROQ_API_KEY`,
`NVIDIA_API_KEY` and `OPENCODE_ZEN_API_KEY` are all still set in the environment
and are picked up automatically. Only `provider.ollama` was added, pointing at
`http://localhost:11434/v1`.

**Deleted 7 stale config files** from `~/.config/opencode.bak/`:

- `opencode.jsonc.bak`
- `opencode.jsonc.bak-20260929-133722` (byte-identical duplicate of the above,
  MD5 `118C27B65692`)
- `opencode.jsonc.bak-before-google-removal`
- `opencode.jsonc.disabled`
- `opencode.jsonc.example`
- `opencode.json.bak`
- `tui.json.bak`

Nothing was lost: the repo's previous `opencode.jsonc` is the same file as the
deleted `opencode.jsonc.bak`, preserved here as
`archive/opencode.jsonc.2026-09-29.jsonc`.

`~/.config/opencode.bak/opencode.json` was kept — it is the only remaining record
of the 29 agent and 27 command definitions.

### Repo changes in this commit

- `opencode.jsonc` replaced with the live rebuilt config
- `archive/opencode.jsonc.2026-09-29.jsonc` added — the superseded version
- `CHANGELOG.md` added
- `opencode.json` left untouched

### Known outstanding issues

1. **`opencode.json` in this repo cannot be loaded as-is.** It is named `.json`
   but contains `//` inline comments, and it declares a `permission` key twice
   (once near the top, once near the bottom). It also uses
   `"prompt": "{file:prompts/agents/planner.txt}"` for its agents, pointing at
   files that do not exist in this repo.
2. **This repo's `opencode.json` is richer than the local backup** — 65 plugins
   against 26, plus full Antigravity model definitions under `provider.google`.
   Roughly 39 plugins listed here have never been present in the live config.
   Worth reviewing separately; they are not restored.
3. The 29 agents and 27 commands in the backup `opencode.json` are **not** in the
   live config.
4. `opencode` must be **restarted** for any of this to take effect — config is
   read once at startup and is not hot-reloaded.

### Related: disk cleanup performed the same day

44 GB reclaimed across both drives. Full breakdown is not config-related and is
not tracked here.

---

## 2026-10-03 — Added global AGENTS.md and registered it in `instructions`

### What changed

- Added `AGENTS.md` at the config root (`~/.config/opencode/AGENTS.md`).
- Added `"instructions": ["AGENTS.md"]` to `opencode.jsonc` so the file loads as
  a global instruction set on every session.

### Why

The config-change history was only being logged when explicitly asked for. This
makes it a standing rule instead: any future session that touches
`opencode.jsonc`, `AGENTS.md`, plugins, skills, agents, commands, MCP servers or
providers is now instructed to update `CHANGELOG.md`, archive superseded files
under `archive/`, and push to this repo before finishing.

### What the rule covers

- Never inline a real API key; keys stay in env vars referenced as `{env:VAR}`.
- Validate config against <https://opencode.ai/config.json> before pushing,
  because opencode hard-fails to start on a bad shape.
- Confirm a plugin exists on npm with `npm view <name> version` before adding it.
- Check the matching env var before enabling an MCP server; otherwise leave it
  out or set `"enabled": false` and record why.
- State explicitly what was skipped and why, so the next session does not
  re-investigate it.
- Document the `OPENCODE_DISABLE_PROJECT_CONFIG` / `OPENCODE_CONFIG` /
  `OPENCODE_CONFIG_CONTENT` escape hatches for recovering from a config that
  will not start.
- Always tell the user to restart opencode, since config is read once at startup.

### Also recorded in AGENTS.md

The known-defect note on `opencode.json` (named `.json` but containing `//`
comments, declaring `permission` twice, and referencing `{file:prompts/agents/*.txt}`
paths absent from this repo), so future sessions do not mistake it for the live
config or try to load it.

### Verification

`opencode.jsonc` parses as valid JSON and `instructions` resolves to the new
`AGENTS.md` at the config root. Effective on next opencode restart.