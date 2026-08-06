# Current setup baseline

Snapshot as of 2026-08-06 (updated same day — design tooling pass). Updated whenever a research
pass finds it's gone stale, or an implemented change makes it inaccurate. This is what each weekly
research pass diffs against — keep it factual and current, not aspirational.

## Skills

**Bundled (platform-provided, not self-managed):** dataviz, artifact-design, artifact-diagramming,
artifact-capabilities, update-config, keybindings-help, simplify, fewer-permission-prompts, loop,
schedule, claude-api, claude-in-chrome, run, init, review, security-review, design-sync,
design-login (the latter two backed by the native `DesignSync` tool — read/write access to
claude.ai/design design-system projects, incremental component-library sync; found during the
2026-08-06 design-tooling pass, not previously documented here — unused so far, no current project
has a claude.ai/design project to sync against).

**Installed plugins:**
- `superpowers@superpowers-dev` (v6.2.0, user scope, github.com/obra/superpowers) — 14 skills:
  brainstorming, dispatching-parallel-agents, executing-plans, finishing-a-development-branch,
  receiving-code-review, requesting-code-review, subagent-driven-development,
  systematic-debugging, test-driven-development, using-git-worktrees, using-superpowers,
  verification-before-completion, writing-plans, writing-skills.
- `playwright@claude-plugins-official` (Microsoft) — browser automation/E2E testing MCP server.
- `claude-code-setup@claude-plugins-official` (v1.0.0) — analyzes a codebase and recommends
  tailored hooks/skills/MCP servers/subagents for it.
- `block-no-verify@claude-code-workflows` (v1.0.1, from `wshobson/agents`) — `PreToolUse` hook
  blocking `--no-verify`/`--no-gpg-sign`/other git-hook-skipping flags.
- `frontend-design@claude-plugins-official` (Anthropic, official) — design-quality skill: steers
  away from "generic AI aesthetics," brainstorm→token-plan→critique→build workflow, accessibility
  floor, copywriting guidance. See `research/2026-08-06-design-ux.md`.
- `modern-web-guidance@claude-plugins-official` (Google Chrome, v0.0.179) — 130+ web-platform
  best-practice checks (CSS/layout, forms, performance, accessibility), local-only/no API key.
- `impeccable@impeccable` (v4.0.4, Paul Bakaus) — 23 design slash-commands
  (polish/audit/critique/harden/typeset/...), companion to `frontend-design`. Ships local-only
  `PostToolUse`/`Stop` hooks on Edit/Write (verified no network calls/exec before install).

**Configured marketplaces:** `claude-plugins-official` (default), `superpowers-dev`
(github.com/obra/superpowers), `claude-code-workflows` (github.com/wshobson/agents), `impeccable`
(github.com/pbakaus/impeccable — added 2026-08-06, design-focused, single plugin).

## Design / UI-UX (added 2026-08-06)

See `research/2026-08-06-design-ux.md` for the full deep-dive. Summary of current state:
- Plugin tooling: `frontend-design` + `modern-web-guidance` + `impeccable` (all above).
- `~/.claude/CLAUDE.md` has a standing "UI/UX & product design" section (accessibility floor,
  screenshot self-critique loop, Nielsen heuristic pass, `frontend-design` for non-trivial UI work)
  — scoped to real UI/product surfaces, not throwaway/internal tooling.
- Design-to-code MCP (Figma/Framelink) evaluated and explicitly declined — no current use case,
  no project's UI originates from design files.
- Known gap, not yet closed: no Compass-specific design-tokens/constraints doc exists yet (the
  thing that would make `frontend-design`'s choices stick consistently project-to-project instead
  of being re-derived each session) — deferred until real Compass UI work is happening, not written
  speculatively. Also: Compass's own UI (`app/globals.css`) is still unstyled `create-next-app`
  scaffold — a real, identified opportunity, not yet acted on (out of scope for a tooling pass, not
  requested).

## Agent architecture / multi-agent tooling

Not previously documented here — added 2026-08-06 after the "agent architectures & multi-agent
patterns" deep-dive research pass found the omission. Three tiers actually available, distinct
from each other:

- **Subagents** (`Agent` tool) — spawned within a session, report a summary back to the caller,
  never talk to each other or share state directly. Named agent types available in this account's
  sessions so far: `claude`, `claude-code-guide`, `Explore`, `general-purpose`, `Plan`,
  `statusline-setup`. Superpowers' `dispatching-parallel-agents` skill operates at this same tier.
- **`Workflow` tool** — deterministic multi-agent orchestration expressed as a script
  (`pipeline()`/`parallel()` primitives, phases, schema-validated structured output). Documented
  idioms include adversarial-verify and judge-panel patterns. Available in this account's sessions;
  not yet used on any real task as of this baseline.
- **Agent Teams** — a Claude Code platform feature (shipped February 2026 with Opus 4.6) where
  multiple full Claude Code instances share a task list with lock-and-claim, message each other
  directly, and one acts as orchestrator. **Experimental and disabled by default**
  (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`), with documented rough edges around session
  resumption, task coordination, and shutdown. Not enabled anywhere; no current task identified
  that needs full-instance teammates rather than subagent fan-out or the `Workflow` tool.

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
- `~/.claude/CLAUDE.md` — created 2026-08-06 (see `decisions/2026-08-06-global-claude-md.md`).
  Short, currently-active standing instructions distilled from this repo's decisions; this repo
  stays the durable reasoning/history. Current sections: token efficiency (explicit Haiku routing
  for cheap/high-volume subagent work; prefer targeted reads over whole-file reads), memory
  discipline.
- Memory system exists (`~/.claude/projects/<project-hash>/memory/`) but is **project-scoped by
  working directory**, not global — a memory saved in one project's session isn't visible in
  another project's session. It was unused (empty) as of this baseline, despite real
  feedback/project facts worth capturing during the Compass session that created this repo — worth
  being more disciplined about using it going forward, project by project. (Reinforced in
  `~/.claude/CLAUDE.md` as of 2026-08-06, not yet re-verified whether it's actually being used more
  since.)

## Browser E2E testing

No `chromium-cli` available in sandboxed/cloud sessions. Compass has a small Playwright-based REPL
(`scripts/browser/`) standing in for it, committed to that project specifically since it's tied to
Compass's own login flow. Not yet generalized into a reusable cross-project tool — worth
revisiting if a second project needs the same thing.

## Sandbox bootstrap notes

Cloud/sandboxed sessions (see Compass's `docs/environments.md` for the fuller version of this
pattern) start with almost nothing installed and no persisted auth — confirmed by having to
install Deno, the Supabase CLI, Playwright, and `gh` from scratch in the session that created this
repo. Specifically for **this repo**: pushing needs `gh auth login` (device-flow: `gh auth login
--hostname github.com --git-protocol ssh --skip-ssh-key`, then `gh auth setup-git`, then use the
`https://` remote URL — the SSH key present in a fresh sandbox is often a repo-scoped deploy key,
e.g. Compass's, not a general-purpose one, so plain SSH push to a *new* repo fails with "Repository
not found" until this is done). A fresh sandbox will need this re-established; a persistent
environment (the user's Mac, the VPS) should only need it once.

## Weekly research routine

`dev-toolkit-weekly-research` (trigger `trig_0177HnwudaaGY532cH8RcxGu`), Saturdays 09:00 UTC, runs
against a fresh cloud checkout of this repo (`sources`), no MCP connectors attached. Two things
learned from its first real run (2026-08-06), both now reflected in the spec:

- **Its cloud sandbox is ephemeral** — no persisted `~/.claude/plugins`, so a skill it decides is
  worth installing can't actually be installed *by it*; the carve-out now has it log a
  recommendation instead (see `decisions/2026-08-06-fix-skills-autonomy-ephemeral-sandbox.md`).
- **Write-access blocker resolved.** The first real run's `git push` and the GitHub MCP write tools
  both failed with "GitHub access is not enabled for this session — an org admin must connect the
  Claude GitHub App." A second run the same day (2026-08-06, this pass) had working `mcp__github__*`
  tools available from the start and `git push origin main` succeeded on the first try, landing
  commit `4c1e0fd` directly — no workaround needed. Whatever needed the GitHub OAuth App
  access-policy approval mentioned in the original blocker note appears to have been granted in the
  meantime. Noting this rather than deleting the old blocker note outright, since this was only
  confirmed working once — if a future scheduled Saturday run hits the same "GitHub access is not
  enabled" error again, treat it as a regression worth investigating, not a first-time surprise.

## Known gaps (as of this baseline)

- No cross-project memory or preference system — only this repo (decisions/reasoning) and
  per-project memory (facts specific to that codebase).
- No local `.mcp.json` used anywhere yet — unclear if/when that's preferable to the platform
  connector system.
- No established testing/verification skill usage pattern prior to installing superpowers
  (`test-driven-development`, `verification-before-completion` now available but unused so far).
- No structured prompting-technique review has been done — current approach is ad hoc.
