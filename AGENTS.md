# Global instructions

## Log every opencode config change to GitHub

This machine's opencode configuration lives at `~/.config/opencode/`. It is
mirrored in the repo <https://github.com/jamesmutegeki/Opencode>, which is the
source of truth and the change log.

**After any change to `opencode.jsonc`, `AGENTS.md`, or anything under
`~/.config/opencode/` — plugins, skills, agents, commands, MCP servers,
providers — commit and push to that repo before finishing.**

Do this:

1. Sync the changed files into a working clone of the repo.
2. Add a dated entry to `CHANGELOG.md` under a `## YYYY-MM-DD — <summary>`
   heading. Cover:
   - what changed and why
   - what was restored, removed, or deliberately left out, **with reasons**
   - anything left in a broken or unverified state
3. Preserve superseded config files under `archive/` rather than deleting them.
4. Commit with a message describing the change, then `git push origin main`.

```powershell
git clone https://github.com/jamesmutegeki/Opencode.git
# ... edit ...
git add CHANGELOG.md opencode.jsonc archive/
git commit -m "..."
git push origin main
```

### Rules

- **Never commit secrets.** API keys stay in environment variables and are
  referenced as `{env:VAR_NAME}` in config. Never inline a real key value.
- **Validate before pushing.** opencode hard-fails to start on invalid config
  (`ConfigInvalidError`). Confirm the file parses and only uses shapes from
  <https://opencode.ai/config.json>.
- **Verify plugins exist** with `npm view <name> version` before adding one.
  An unresolvable plugin entry breaks startup.
- **Check API keys before enabling an MCP server.** If the matching env var is
  unset, either leave the server out or add it as `"enabled": false` and note it
  in the changelog.
- **Say what was skipped.** Omitting a decision silently makes the next session
  re-investigate it.

### Verify a config actually loads

After changing config, confirm opencode still starts cleanly before pushing.
If it will not start, the escape hatches are:

- `OPENCODE_DISABLE_PROJECT_CONFIG=1` — skip project config, load globals only
- `OPENCODE_CONFIG=/path/to/file.json` — load an explicit config
- `OPENCODE_CONFIG_CONTENT='{"$schema":"https://opencode.ai/config.json"}'` —
  inject inline config as a final merge

A running session keeps using the config it loaded at startup. **Always tell the
user to restart opencode after a config change.**

## Known repo issues

`archive/opencode.json.archived-2026-10-17` (formerly the repo-root
`opencode.json`) is an archive of agent/command/plugin definitions and is
**not** loadable as-is: it is named `.json` but contains `//` inline comments,
declares the `permission` key twice, and its agents reference
`{file:prompts/agents/*.txt}` paths that do not exist in the repo. It was moved
into `archive/` on 2026-10-17 because the home directory is a checkout of this
repo, and a root-level `opencode.json` materializes there, where opencode loads
it as project config — the cause of the 2026-10-15 startup crash. Never place a
file named `opencode.json` at the repo root. The live config is `opencode.jsonc`.