# Current setup baseline

Snapshot as of 2026-08-06. Updated whenever a research pass finds it's gone stale, or an
implemented change makes it inaccurate. This is what each weekly research pass diffs against —
keep it factual and current, not aspirational.

## Skills

**Bundled (platform-provided, not self-managed):** dataviz, artifact-design, artifact-diagramming,
artifact-capabilities, update-config, keybindings-help, simplify, fewer-permission-prompts, loop,
schedule, claude-api, claude-in-chrome, run, init, review, security-review.

**Installed plugins:**
- `superpowers@superpowers-dev` (v6.2.0, user scope, github.com/obra/superpowers) — 14 skills:
  brainstorming, dispatching-parallel-agents, executing-plans, finishing-a-development-branch,
  receiving-code-review, requesting-code-review, subagent-driven-development,
  systematic-debugging, test-driven-development, using-git-worktrees, using-superpowers,
  verification-before-completion, writing-plans, writing-skills.

## MCP servers / integrations

Connected via the platform's own connector system (claude.ai account-level), not local
`.mcp.json` — no such file exists in any project checked so far. Currently connected:
Supabase, Wallet (BudgetBakers), Gmail, Google Calendar, Google Drive. Adding a new one requires
the user to connect it via the claude.ai UI, not something installable from a session directly —
unless a project specifically wants a local, project-scoped MCP server via `.mcp.json`, which is a
different (also available) mechanism not currently used anywhere.

## Global config (`~/.claude/`)

- `settings.json`: statusline command, dark theme, `superpowers@superpowers-dev` enabled,
  `agentPushNotifEnabled: true`. No custom permission allowlists, no custom hooks beyond what the
  superpowers plugin adds (one `SessionStart` hook).
- No global `CLAUDE.md`.
- Memory system exists (`~/.claude/projects/<project-hash>/memory/`) but is **project-scoped by
  working directory**, not global — a memory saved in one project's session isn't visible in
  another project's session. It was unused (empty) as of this baseline, despite real
  feedback/project facts worth capturing during the Compass session that created this repo — worth
  being more disciplined about using it going forward, project by project.

## Browser E2E testing

No `chromium-cli` available in sandboxed/cloud sessions. Compass has a small Playwright-based REPL
(`scripts/browser/`) standing in for it, committed to that project specifically since it's tied to
Compass's own login flow. Not yet generalized into a reusable cross-project tool — worth
revisiting if a second project needs the same thing.

## Known gaps (as of this baseline)

- No cross-project memory or preference system — only this repo (decisions/reasoning) and
  per-project memory (facts specific to that codebase).
- No local `.mcp.json` used anywhere yet — unclear if/when that's preferable to the platform
  connector system.
- No established testing/verification skill usage pattern prior to installing superpowers
  (`test-driven-development`, `verification-before-completion` now available but unused so far).
- No structured prompting-technique review has been done — current approach is ad hoc.
