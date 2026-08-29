# Current setup baseline

Snapshot as of 2026-08-06, last updated 2026-08-29 (tooling/infrastructure/automation weekly pass —
noted the bundled skill roster had drifted from what was documented here, most notably
`session-start-hook` as an unaddressed fix for the sandbox-bootstrap pain point, and updated
marketplace plugin counts). Updated whenever a research pass finds it's gone stale, or an
implemented change makes it inaccurate. This is what each weekly research pass diffs against — keep
it factual and current, not aspirational.

## Skills

**Bundled (platform-provided, not self-managed):** dataviz, artifact-design, artifact-diagramming,
artifact-capabilities, update-config, keybindings-help, simplify, fewer-permission-prompts, loop,
schedule, claude-api, claude-in-chrome, run, init, review, security-review, design-sync,
design-login (the latter two backed by the native `DesignSync` tool — read/write access to
claude.ai/design design-system projects, incremental component-library sync; found during the
2026-08-06 design-tooling pass, not previously documented here — unused so far, no current project
has a claude.ai/design project to sync against). Also present in the 2026-08-29 run's session (not
previously listed here — the bundled roster is evidently larger/more session-dependent than this
list captured): `session-start-hook` (creates a repo's `SessionStart` hook so cloud/web sessions can
run tests/linters without manual setup — directly relevant to the "Sandbox bootstrap notes" pain
point below, see `research/2026-08-29-tooling-infrastructure-automation.md`), `design` (canvas/
artboard design tool, distinct from `design-sync`/`design-login`), `workflow-authoring` (the
reference skill backing the `Workflow` tool, see "Agent architecture" below), `docx`,
`import-memory`, `morning`, `pdf`, `pptx`, `skill-creator`, `unslop`, `xlsx`. That same session
did *not* show `schedule`, `claude-in-chrome`, `design-sync`, `design-login`, or `review` (it
showed `code-review` instead) — plausibly connector-/feature-gated absences specific to that
ephemeral cloud sandbox rather than actual removals, but not independently confirmed; worth
reconfirming from a normal persistent-environment session, including whether `review` was
genuinely renamed to `code-review` (several `skills-seen.md` rejection rows cite `review` by that
name and would need a corresponding update if so).

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

**Configured marketplaces:** `claude-plugins-official` (default, 291 plugins as of 2026-08-29, up
from 286), `superpowers-dev` (github.com/obra/superpowers), `claude-code-workflows`
(github.com/wshobson/agents, 93 plugins as of 2026-08-29, up from 92), `impeccable`
(github.com/pbakaus/impeccable — added 2026-08-06, design-focused, single plugin), `thedotmack`
(github.com/thedotmack/claude-mem — added 2026-08-08, 2 plugins as of 2026-08-29 —
`claude-mem`/`claude-mem-cowork` — both rejected, see Known gaps). Also evaluated and declined:
`tonsofskills.com` (`jeremylongshore/tons-of-skills-marketplace`, 471 plugins/3,069 skills, same
maintainer/project as the already-rejected `claude-code-plugins-plus-skills`, just rebranded —
same infeasible-vetting-at-scale reasoning, not added).
Note: the weekly routine's cloud sandbox is ephemeral, so none of these persist between runs; each
run re-adds them from this list to inspect current listings (see "Weekly research routine" below).

**Recommended, not yet installed (pending next manual review):** `context7` (2026-08-08,
library-documentation-lookup MCP server), `chrome-devtools-mcp` and `typescript-lsp` (2026-08-15,
live browser debugging and TypeScript language-server integration respectively — see
`decisions/2026-08-15-install-chrome-devtools-mcp.md` and
`decisions/2026-08-15-install-typescript-lsp.md`).

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

## Testing / verification

Added 2026-08-22 after the "testing/verification approaches" deep-dive found the omission (this
topic previously only appeared as a line in Known gaps, not its own section, unlike the other five
rotation topics). Current state:

- **Skill-level:** superpowers' `test-driven-development`, `verification-before-completion`, and
  `systematic-debugging` — installed since 2026-08-06, still not exercised on a real task as of
  this baseline (same "installed, unused" note carried forward from the prior snapshot).
- **`Workflow`-tool-level:** documented adversarial-verify and judge-panel idioms (independent
  skeptic subagents that try to refute a finding; multi-persona panels for ambiguous judgment
  calls) — available, **still not used on any real task** as of this baseline. This is now the
  second consecutive deep-dive pass noting this same "documented, zero real-task usage" gap; see
  `research/2026-08-22-testing-verification-approaches.md` for the explicit flag not to just let
  this line get copy-pasted forward a third time without someone actually trying it.
- **Native `/goal` command** (Claude Code v2.1.139+, May 2026) — not previously documented here as
  a verification mechanism in its own right (baseline only mentioned it in passing as the reason
  `ralph-wiggum` was rejected). Declares a completion condition; after every turn, a separate fresh
  evaluator model (defaults to Haiku) checks that condition against the conversation and returns
  yes/no + reason, independent of the model doing the work. No install required. Limitation: the
  evaluator only judges what's already in the conversation — it doesn't independently re-run
  commands or read files itself, so it complements but doesn't replace an actual test/build run.
  Distinct from `Stop` hooks, which are settings/team-scoped rather than session-scoped and *can*
  shell out to real checks.
- **Bundled skills:** `review`, `security-review` cover code-review-shaped verification; no
  dedicated eval/benchmark framework installed (`deepeval`, `codspeed` both evaluated and rejected
  — see `skills-seen.md` — no project currently runs an eval harness against a shipped AI feature).

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
- **Auto memory** (Claude's own accumulated notes — distinct from `CLAUDE.md`, which is
  user-written) lives at `~/.claude/projects/<project>/memory/`, keyed by git repo (shared across
  worktrees/subdirectories of the same repo), on by default (`autoMemoryEnabled`), with a
  `MEMORY.md` entrypoint (200-line/25KB load cap, self-curated) plus on-demand topic files.
  **Project-scoped**, not global — a memory saved in one project's session isn't visible in
  another's. It was unused (empty) as of the 2026-08-06 baseline; not yet re-verified whether it's
  actually being used more since being reinforced in `~/.claude/CLAUDE.md`.
- **Subagents can have their own separate memory** via a `memory: user|project|local` field in a
  custom subagent's frontmatter (`.claude/agents/*.md`) — found during the 2026-08-15 deep-dive
  (`research/2026-08-15-memory-systems.md`). Depends on the session's own auto memory being enabled
  (no independent toggle). **Not currently actionable**: no project tracked here defines any custom
  subagents yet — this account's available agent types (`claude`, `claude-code-guide`, `Explore`,
  `general-purpose`, `Plan`, `statusline-setup`) are all platform-provided, not project-defined.
  Revisit when a project has a repeatedly-invoked custom subagent worth giving persistent memory to.
- **`.claude/rules/`** (and user-level `~/.claude/rules/`) — path-scoped instruction files, distinct
  from both `CLAUDE.md` and auto memory: markdown files that load into context only when Claude
  reads a file matching their `paths:` frontmatter glob, instead of loading at every session start.
  Found 2026-08-15; not yet used anywhere. See
  `decisions/2026-08-15-token-efficiency-path-scoped-rules.md` for the drafted recommendation (not
  yet applied) on when to use this for `~/.claude/CLAUDE.md` itself as it grows.

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

**Candidate fix, not yet applied:** the bundled `session-start-hook` skill (noticed 2026-08-29, see
"Skills" above) exists specifically to turn manual bootstrap steps like these into a real
`SessionStart` hook instead of a doc a human/agent has to follow by hand each time. Worth trying on
Compass first (`docs/environments.md` already documents the steps to convert) before considering it
for this repo's own smaller bootstrap needs.

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
- **Still holding as of 2026-08-15** — `git push origin main` succeeded on the first try again, no
  workaround needed. Third consecutive successful run since the fix.
- **Still holding as of 2026-08-22** — fourth consecutive successful push, first try, no
  workaround. This note can likely be retired soon if it keeps holding, but kept for now per the
  same "note, don't delete yet" logic.
- **Still holding as of 2026-08-29** (this run) — fifth consecutive successful push, first try, no
  workaround.

## Known gaps (as of this baseline)

- No cross-project memory or preference system — only this repo (decisions/reasoning) and
  per-project memory (facts specific to that codebase). Best candidate found so far to close this,
  `claude-mem` (88.5k★), was still rejected on 2026-08-08 — its automatic-hooks footprint is larger
  than the two previously-rejected memory tools, not smaller; would need an actual read of its
  worker-service source before it clears the bar. See `research/2026-08-08-prompting-techniques.md`.
- No local `.mcp.json` used anywhere yet — unclear if/when that's preferable to the platform
  connector system.
- ~~No established testing/verification skill usage pattern prior to installing superpowers~~ — now
  has a dedicated "Testing / verification" section above (added 2026-08-22); the underlying gap
  itself (skills/tools installed and documented, still not exercised on a real task) remains open,
  just tracked there now instead of here.
- ~~No structured prompting-technique review has been done~~ — done 2026-08-08
  (`research/2026-08-08-prompting-techniques.md`): found current practice (superpowers'
  plan/execute/verify skills, this repo's own `specs/` workflow, subagent-dispatch patterns) already
  substantially aligned with 2026 SOTA (spec-driven development, PLAN/ACT/VERIFY/ITERATE framing,
  meta-prompting/step-back decomposition) — a confirming pass, not a gap-closing one. One concrete,
  actionable finding came out of it (effort-level routing for `Workflow` agent dispatch,
  independent of the existing model-routing rule) but it's filed under token-efficiency, not here,
  since it's a delegation-pattern lever rather than a prompt-wording one.
