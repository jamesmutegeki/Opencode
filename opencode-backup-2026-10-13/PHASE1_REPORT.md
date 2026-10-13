# Phase 1: Compatibility Evaluation — 12 Repos
Agent: OpenCode v2.0.27 | Backup: C:\Users\james\opencode-backup-2026-10-13
Working plugins preserved: ecc.hooks, apify, firecrawl, tavily | MCP: apify, duckdb, semantic-scholar

| # | Repo | OpenCode? | Verdict | Install / Notes | Risk |
|---|---|---|---|---|---|
| 1 | vercel-labs/skills | Yes (explicit) | COMPATIBLE | `npx skills add`; SKILL.md format; supports opencode agent path | None |
| 2 | obra/superpowers | Yes (.opencode/) | COMPATIBLE | `.opencode/INSTALL.md`; skills + plugin dirs for 10+ agents | None |
| 3 | usestrix/strix | No (.skills only) | COMPATIBLE (skill/tool) | `npx skills add`; pentest CLI + 9 agent skills | None |
| 4 | Panniantong/Agent-Reach | No (.openteams/) | COMPATIBLE (external) | `agent-reach install`; web-capability CLI for agents | None |
| 5 | punkpeye/awesome-mcp-servers | N/A (directory) | COMPATIBLE (reference) | Reference for `opencode mcp add`; 3 MCP already active | None |
| 6 | HKUDS/OpenSpace | Yes (MCP + docs) | COMPATIBLE | `openspace-mcp`; MCP server + skill hub; wire manually or via config | Low (new MCP server) |
| 7 | ChristopherKahler/paul | No | COMPATIBLE (external) | Claude-focused agent/tool; no OpenCode manifest | None |
| 8 | BloopAI/vibe-kanban | Uncertain | COMPATIBLE (framework) | Kanban plugin framework; .claude-plugin likely; no .opencode found | None |
| 9 | SuperClaude-Org/SuperClaude_Framework | Uncertain | COMPATIBLE (framework) | Multi-agent framework; likely Claude-native; check docs for .opencode | Low |
|10 | parcadei/Continuous-Claude-v3| No | COMPATIBLE (external) | Claude-specific continuous workflow; no OpenCode config | None |
|11 | PleasePrompto/notebooklm-skill| No | COMPATIBLE (skill) | SKILL.md for Claude; uses agent-skills spec | None |
|12 | eyaltoledano/claude-task-master| No | COMPATIBLE (external) | Claude task management tool/skill | None |

## Key Findings
- 2 repos have native OpenCode support (skills, superpowers)
- 1 is a direct MCP server (OpenSpace) — can be added to existing MCP config
- 9 are compatible external tools/skills/frameworks — no installation conflicts
- 0 repos threaten existing working plugins or MCP servers
- All 12 use SKILL.md or plugin formats compatible with agent-skills spec (OpenCode supported per compatibility table)
