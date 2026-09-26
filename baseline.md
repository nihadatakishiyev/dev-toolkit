# Current setup baseline

Snapshot as of 2026-08-06, last updated 2026-09-26 (memory-systems weekly pass — refreshed
`claude-plugins-official`'s plugin count, added native `AGENTS.md` support to Global config, and
extended the "Weekly research routine" push-confirmation streak). Updated whenever a research pass
finds it's gone stale, or an implemented change makes it inaccurate. This is what each weekly research
pass diffs against — keep it factual and current, not aspirational.

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

**Configured marketplaces:** `claude-plugins-official` (default, 314 plugins as of 2026-09-26, up
from 310 — a handful of the growth individually itemized (`duckdb-skills`, `gitkraken`, `expo`,
`convex`, `mlflow`, `postman`, all rejected), the rest folding into the same established
bulk-reject categories (SaaS/credentialed integrations, cloud-provider suites, unused-language/
framework tooling) as the 2026-08-08/2026-08-29/2026-09-19 passes, see
`research/2026-09-26-memory-systems.md`), `superpowers-dev` (github.com/obra/superpowers),
`claude-code-workflows` (github.com/wshobson/agents, 94 plugins, unchanged since 2026-09-05),
`impeccable` (github.com/pbakaus/impeccable — added 2026-08-06, design-focused, single plugin),
`thedotmack` (github.com/thedotmack/claude-mem — added 2026-08-08, 2 plugins —
`claude-mem`/`claude-mem-cowork` — both rejected, see Known gaps), `claude-plugins-community`
(`claude-community`, github.com/anthropics/claude-plugins-community — added 2026-09-12, first-party
Anthropic-run **community** marketplace, distinct from `claude-plugins-official`: anyone submits a
plugin, it passes automated security scanning, and it's synced in nightly — 2,282 plugins, unchanged
since 2026-09-12 (first pass where this marketplace hasn't grown), far larger than anything else
configured, and reputable only as a *host*; individual plugins there still need the same per-plugin
reputability vetting as any third-party source, since passing an automated security scan is a
materially weaker bar than "well-established/maintained," see
`research/2026-09-12-agent-architectures-multi-agent-patterns.md`). Also evaluated and declined:
`tonsofskills.com` (`jeremylongshore/tons-of-skills-marketplace`, 471 plugins/3,069 skills, same
maintainer/project as the already-rejected `claude-code-plugins-plus-skills`, just rebranded — same
infeasible-vetting-at-scale reasoning, not added), `davepoon/buildwithclaude` (3.4k★, single
maintainer, indexes 20k+ community plugins with no independent per-entry vetting — same
infeasible-vetting-at-scale reasoning despite the higher star count, not added), `AqueGen/model-routing`
(2026-09-19 — pinned model-tier subagents + routing skill, directly on-topic for the Haiku-routing
practice below, but only 3★ and days-old at evaluation time — too thin a track record, revisit if it
matures), `netresearch/claude-code-marketplace` (2026-09-26 — well-maintained, company-backed
(Netresearch DTT GmbH), 61★/14 forks/366 commits, but all 39 skills concentrate on TYPO3/PHP/Go/
Docker/Jira, a stack no tracked project uses — not added; notable for using the cross-tool
`agentskills.io`/`AGENTS.md` portability standard, see this week's memory-systems deep-dive).
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
  Separately, `subagent_type: "fork"` (found 2026-09-05, see
  `research/2026-09-05-mcp-servers-integrations.md`) is a distinct mode, not a named type: it
  inherits the entire parent conversation (system prompt, tools, model — always the parent's model,
  the `model` override is ignored for it — full history) instead of starting cold, reusing the
  parent's prompt cache on its first request. Cheaper than a fresh subagent when the subtask
  genuinely needs prior context; worse than one for independent/adversarial verification or when
  the parent conversation is mostly irrelevant bulk. See
  `decisions/2026-09-05-token-efficiency-fork-subagent.md` (drafted recommendation, not yet applied
  to `~/.claude/CLAUDE.md`).
- **`Workflow` tool** — deterministic multi-agent orchestration expressed as a script
  (`pipeline()`/`parallel()` primitives, phases, schema-validated structured output). Documented
  idioms include adversarial-verify and judge-panel patterns. Available in this account's sessions;
  not yet used on any real task as of this baseline.
- **Agent Teams** — a Claude Code platform feature (shipped February 2026 with Opus 4.6) where
  multiple full Claude Code instances share a task list with lock-and-claim, message each other
  directly, and one acts as orchestrator. **Experimental and disabled by default**
  (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`), with documented rough edges around session
  resumption, task coordination, and shutdown. Not enabled anywhere; no current task identified
  that needs full-instance teammates rather than subagent fan-out or the `Workflow` tool. Still not
  GA as of 2026-09-12 (independently reconfirmed via web search — "research preview," experimental
  flag still required).

**Anthropic's own published taxonomy** (found 2026-09-12, see
`research/2026-09-12-agent-architectures-multi-agent-patterns.md`) names five coordination
patterns: Orchestrator-Subagent, Generator-Verifier, Agent Teams, Message Bus, and Shared State.
The three tiers above map onto three of the five — Orchestrator-Subagent = the `Agent` tool (the
recommended default, and already this account's primary pattern), Generator-Verifier = the
`Workflow` tool's adversarial-verify idiom (available, still not exercised on a real task — see
Testing/verification below), Agent Teams = the platform feature above. **Message Bus**
(event-driven pipeline of pluggable agents) and **Shared State** (blackboard-style collaboration
over literally shared mutable state) have no current analog or identified need in any tracked
project — noted for completeness, not an action item.

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
  third consecutive pass to note this same "documented, zero real-task usage" gap since it was
  first flagged (2026-08-22 original, reinforced 2026-09-12 via the Generator-Verifier mapping,
  reinforced again 2026-09-19) — genuinely due for a forced try next time a verification-shaped
  task comes up rather than letting it ride a fourth time; see
  `research/2026-08-22-testing-verification-approaches.md` for the original flag.
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

Three distinct mechanisms identified so far (last two both added to this doc 2026-09-05, see
`research/2026-09-05-mcp-servers-integrations.md`):

- **Platform connector system** (claude.ai account-level), not local `.mcp.json` — no such file
  exists in any project checked so far. Currently connected: Supabase, Wallet (BudgetBakers),
  Gmail, Google Calendar, Google Drive. Adding a new one requires the user to connect it via the
  claude.ai UI, not something installable from a session directly. A native `SearchMcpRegistry`
  tool can search this same account-level directory by keyword from within a session (returns
  install state, sample tools, whether enabled in the current chat) — it doesn't connect anything
  itself, but lets a session proactively surface a specific real connector by name instead of only
  reacting once a user names one.
- **Project-scoped `.mcp.json`** — a different, also-available mechanism; not currently used
  anywhere.
- **Session/environment-level MCP tools** — pre-configured for a specific session by the execution
  environment itself, not attached by the user and not a project file. This weekly routine's own
  cloud session is an example: it gets repo-scoped `mcp__github__*` tools supplied directly by the
  remote execution environment, distinct from both mechanisms above (scoped to which repos that
  particular session was granted, not an account-wide toggle or a file in the repo).

Also noted 2026-09-05: the MCP protocol itself had a substantial spec revision (2026-07-28) —
`sampling`/`roots`/SSE-based logging deprecated in favor of a multi-round-trip elicitation
mechanism, plus new "MCP Apps" (tools returning interactive HTML UI in a sandboxed iframe). Doesn't
change anything about how MCP is used here (nothing here authors/hosts MCP servers), but is a
useful freshness signal for future server evaluations — visible adoption of the new spec vs.
legacy long-lived-SSE patterns.

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
- **`AGENTS.md` native support** (Claude Code v2.1.277+, found 2026-09-26 during the memory-systems
  deep-dive's revisit — not in the original 2026-08-15 pass) — Claude Code can now read a
  repository's `AGENTS.md` directly as project instructions (build/test commands, conventions) when
  no `CLAUDE.md`/`CLAUDE.local.md` exists on the path, no import/symlink workaround required; a
  `/config` → "Project instructions" setting controls whether `CLAUDE.md`/`AGENTS.md` are read
  together, separately, or not at all. A companion `/import` command (v2.1.213+) one-time-copies
  another coding tool's config (Cursor rules, `.github/copilot-instructions.md`, etc.) into the
  matching `CLAUDE.md`. **Not currently actionable**: no project tracked here uses `AGENTS.md` or
  another AI coding tool alongside Claude Code, so there's no concrete driver to adopt it yet —
  worth revisiting if a tracked project ever needs instructions portable across multiple AI coding
  tools (this is the same portability angle `netresearch/claude-code-marketplace`'s skills lean on —
  see `research/2026-09-26-memory-systems.md`).

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
- **Still holding as of 2026-08-29** — fifth consecutive successful push, first try, no
  workaround.
- **Still holding as of 2026-09-05** — sixth consecutive successful push, first try, no
  workaround. This note has now held for a full month across six runs; per the "note, don't delete
  yet" logic it'll keep being carried forward, but at this point a regression would be the
  surprising outcome, not the other way around.
- **Still holding as of 2026-09-12** — seventh consecutive successful push, first try, no
  workaround.
- **Still holding as of 2026-09-19** — eighth consecutive successful push, first try, no
  workaround. This note is now carrying forward mostly as a formality; a regression would be
  the surprising outcome.
- **Still holding as of 2026-09-26** (this run) — ninth consecutive successful push, first try, no
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
  since it's a delegation-pattern lever rather than a prompt-wording one. **Revisited 2026-09-19**
  (`research/2026-09-19-prompting-techniques.md`), same confirming verdict, plus one new piece of
  vocabulary worth recording: Anthropic has publicly named **"loop engineering"** — persistent
  instructions/skills/hooks that make an agent iterate autonomously (gather context → act → verify →
  correct) until a defined completion condition holds, built from primitives already documented
  separately here (`CLAUDE.md`, skills, hooks, the native `/goal` command, `/loop`/`/schedule`
  routines). This weekly research routine *is* a loop-engineered system by this definition — the
  term wasn't previously connected to it. Adjacent to, but distinct from, the still-unexercised
  Generator-Verifier idiom below (iterate-until-done vs. produce-then-independently-check); no
  behavior change indicated, just terminology worth citing next time this topic comes up.
