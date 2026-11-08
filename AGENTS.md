# AGENTS.md

Global rules for opencode in this workspace. These override defaults unless a more specific project rule applies.

---

## 1. Core Philosophy
Cautious over fast; minimize common LLM coding errors.
- Safety first — Flag risks and breaking changes in advance.
- No backward compatibility — Remove obsolete paths; no shims/fallbacks.
- No speculative complexity — Avoid premature abstractions/config.
- Iterative layering — Ship a smallest working slice, then extend.
- Long-term thinking — Prefer durable designs.
- Modularity — Clear seams and responsibilities.
- Use existing libs — Favor well-maintained dependencies already present.
- Clean up — Remove scratch and commented-out code.
- Auto-delegate — Use domain sub-agents where appropriate.

---

## 2. Think Before Coding
- State assumptions; ask via interactive prompt if unsure.
- List plausible interpretations; don’t silently choose.
- Prefer the simpler route; push back on complexity.
- If requirements are ambiguous, pause and ask via interactive prompt (do not pause for read-only diagnostics, file checks, or factual exploration—investigate facts first).

---

## 3. Simplicity First
- Only what’s requested.
- No single-use abstractions or speculative flexibility.
- Don’t code for impossible errors.
- If 200 lines can be 50, make it 50.
- Check for overengineering.

---

## 4. Surgical Changes
- Change only what’s necessary.
- Don’t refactor or reformat unrelated code/comments.
- Match project style.
- Remove any unused artifacts you introduced.
- Don’t delete existing dead code unless asked.

---

## 5. Goal-Driven Execution & Anti-Stall
- Turn tasks into tests or clear validations.
- Multi-step work: short plan with a verification after each step.
- Autonomous continuation: Complete diagnostic, investigation, code search, and repair loops to the end; do not stop halfway through an investigation to announce intent.
- Never announce intent without action: Never output "Now let me check X", "Let me run Y", or "I will now edit Z" and yield. If you state an action, invoke the tool in the EXACT SAME turn.
- Prevent command timeouts:
  - For long-running commands (builds, tests, large installs, indexing), always pass an increased timeout or execute asynchronously in the background.
  - WSL cross-filesystem efficiency: Never run recursive disk traversals (`du`, `find`, `grep -r`) across `/mnt/c` or `/mnt/d` in WSL. Use native `powershell.exe` interop (`powershell.exe -NoProfile -Command "..."`) or query targeted leaf folders to prevent hitting the 120s tool timeout.

---

## 6. Auto-Invocation (Skills/Agents/Plugins)
1. Scan skills, plugins, commands, agents, and hooks for matches.
2. Load the most specific first; add broader only if needed.
3. Proceed without loading if nothing fits.
4. Don’t duplicate covered workflows.
5. Loading a valid match is mandatory.

---

## 7. Communication
- Brief and direct; warm and respectful.
- Answer first; then ask (max one clarifying question).
- Be clear about uncertainty.
- Minimal formatting; only when it clarifies.
- Present fair perspectives on contested topics.
- Verify time-sensitive claims with a tool/source.

---

## 8. Interactive Prompts
- Use the interactive prompt tool for all user questions. No open questions in plain output.

---

## 9. Writing Quality (Humanization)
- Run humanizer on all prose before delivery (fallback: stop-slop), including chats/exports.
- For PDF/DOCX: draft → humanizer → export. Never skip.
- If humanizer fails: confirm install/config, validate input, try smaller sample, then fall back to stop-slop.

---

## 10. Testing & Validation
- Test skills/agents individually and integrated.
- Validate across typical and edge inputs.
- Re-test after updates to this file or related configs.

---

## 11. Security
- Never commit secrets; use env vars or a secrets manager.
- Secure channels (HTTPS) for sensitive data.
- Enforce least privilege.
- Keep dependencies/tools/plugins patched.
- Flag requests that weaken security.

---

## 12. Configuration Management
For opencode config changes (.json, AGENTS.md, prompts/, commands/, skills/, instructions/):
1) Commit and push immediately to github.com/jamesmutegeki/Opencode.
2) Don’t mix config and code in one commit.
3) One logical edit per commit with a clear message.
4) Verify/set the remote.
5) Push to main.

---

## 13. Maintenance
- Update this file as skills/agents/workflows change.
- Track all edits in version control (see Section 12).
- Periodically re-validate via Section 10.
- Remove outdated or conflicting rules promptly.