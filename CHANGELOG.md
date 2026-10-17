# Changelog

Running index of changes actually implemented (not just researched/proposed). Newest first, each
entry links to its full `decisions/` record.

## 2026-10-17

- **Thirteenth weekly research pass: MCP servers & integrations deep-dive (rotation's 1st topic,
  starting lap 3).** Skills scan was quiet again — both large first-party marketplaces
  (`claude-plugins-official` at 315, `claude-code-workflows` at 94) showed zero net growth;
  `claude-community` **shrank** 2284 → 2283, its first observed drop since being added
  2026-09-12; no new marketplace found; no plugin recommended. Token-efficiency check found nothing
  new clearing the quality bar — confirming pass. The substantive finding was for `baseline.md`
  itself: calling the native `ListConnectors` tool directly (ground truth, not inference) found the
  documented connector list was stale — `Google Calendar`/`Google Drive` are currently
  **not connected** (baseline said they were), and two connectors weren't documented at all
  (`Compassfi`, connected; `Scalable Capital`, auth incomplete) — corrected in place. Also
  documented, for the first time, that there are **three distinct skill/plugin catalogs** rather
  than one: Claude Code's own CLI plugin-marketplace system (what this routine has always tracked),
  a separate claude.ai account-level plugin catalog (currently empty/unused, queried via native
  `ListPlugins`/`SearchPlugins` tools), and claude.ai's personal Skills library (queried via
  `ListSkills`/`SearchSkills`, a narrower subset that excludes every Claude-Code-specific bundled
  skill). See
  [`research/2026-10-17-mcp-servers-integrations.md`](research/2026-10-17-mcp-servers-integrations.md).

## 2026-10-10

- **Twelfth weekly research pass: tooling/infrastructure/automation deep-dive (rotation's 6th
  topic, completing one full lap — MCP servers next week).** Skills scan was genuinely quiet: both
  large first-party marketplaces (`claude-plugins-official` at 315, `claude-code-workflows` at 94)
  showed zero net growth, `claude-community` grew by only +1 (too marginal to isolate by name);
  no new marketplace found via web search; no plugin recommended. Found three new Claude Code
  mechanisms from this week's v2.1.287–v2.1.294 release burst: **Claude Mods** (in-process,
  unsandboxed plugin handlers distinct from skills/hooks/MCP — added to `baseline.md`, no current
  use case), a native **`effort` parameter on the plain `Agent` tool** (corrects a stale claim in
  `decisions/2026-08-08-token-efficiency-effort-routing.md` that it didn't exist — the tool's own
  built-in restriction to explicit-ask-only is already the right boundary, no further action
  needed), and the opt-in **"You Should Know"** side-agent (requires enabling telemetry — logged as
  a research candidate for manual review, not drafted as a recommendation). **Recommended** (not
  yet applied): a narrow boundary clause on the existing Haiku-routing token-efficiency bullet —
  Claude Haiku 5.5 (now the Anthropic API's default Haiku model) carries a 5x price multiplier past
  100K prompt tokens, which can silently erase the routing bullet's cost rationale for a
  large-prompt dispatch. See
  [`research/2026-10-10-tooling-infrastructure-automation.md`](research/2026-10-10-tooling-infrastructure-automation.md)
  and
  [`decisions/2026-10-10-token-efficiency-haiku-pricing-cliff.md`](decisions/2026-10-10-token-efficiency-haiku-pricing-cliff.md).

## 2026-10-03

- **Eleventh weekly research pass: testing/verification deep-dive (rotation's 5th topic,
  revisited).** Substantive finding: **`claude plugin eval`** (Claude Code v2.1.269+, confirmed
  live in this session's v2.1.288) — a native command for scoring a plugin's own behavior against
  an `evals/` directory of prompt+grader cases, with-arm-vs-without-arm delta scoring, six grader
  types, and CI gating flags. Added to `baseline.md`'s Testing/verification section; not yet applied
  to a real scan, but directly relevant to this routine's own skills-scan job (an actual measured
  signal instead of star-count/maintainer-reputation guessing for borderline plugin candidates).
  Also noted: the `Workflow` tool's adversarial-verify/judge-panel idioms are now on their fourth
  consecutive mention as "documented, zero real-task usage" — flagged as overdue for a manual-review
  pass to actually exercise, not just re-noted again next rotation. Skills scan: `claude-plugins-
  official` grew 314 → 315 (net), chipped away at the long-standing unitemized bulk tail (8 names
  individually evaluated, ~204 bulk-rejected); found and evaluated (not added)
  `ComposioHQ/awesome-claude-plugins` (reputable but every plugin redundant/credential-gated) and
  `alirezarezvani/claude-skills` (27.3k★ — highest star count seen yet, but same
  infeasible-vetting-at-scale verdict as `davepoon/buildwithclaude`). No plugin or token-efficiency
  technique cleared the recommend bar this week — no `decisions/` entries drafted. See
  [`research/2026-10-03-testing-verification-approaches.md`](research/2026-10-03-testing-verification-approaches.md).

## 2026-09-26

- **Tenth weekly research pass: memory systems deep-dive (rotation's 4th topic, revisited).**
  Found and added to `baseline.md`: Claude Code now natively reads a repository's `AGENTS.md` as
  project instructions (v2.1.277+, no import/symlink workaround needed), with a companion
  `/import` command for pulling in another coding tool's config — the single substantive memory-
  system change since the first 2026-08-15 pass. Reconfirmed (again) that "Dreaming"/memory
  consolidation belongs to the separate Managed Agents API, not Claude Code's own auto memory —
  search results keep conflating the two. Checked Anthropic's Context Editing API + Memory Tool
  (Messages-API-level, not applicable to any tracked project). Individually evaluated five
  genuinely new `claude-plugins-official` plugins (`duckdb-skills`, `gitkraken`, `expo`, `convex`,
  `mlflow`, marketplace grew 310 → 314) — all rejected, no mapped workflow or requires a
  credential. Found and evaluated (not added) `netresearch/claude-code-marketplace` — well-
  maintained but 100% concentrated on a stack (TYPO3/PHP/Go/Docker/Jira) no tracked project uses;
  notable for using the `agentskills.io`/`AGENTS.md` cross-tool portability standard, tying
  directly to the `AGENTS.md` finding above. No plugin or token-efficiency technique cleared the
  recommend bar this week — no `decisions/` entries drafted. See
  [`research/2026-09-26-memory-systems.md`](research/2026-09-26-memory-systems.md).

## 2026-09-19

- **Ninth weekly research pass: prompting techniques deep-dive (rotation's 3rd topic,
  revisited).** Confirming pass, consistent with the 2026-08-08 first-pass verdict — no
  prompting-technique gap found. One new piece of vocabulary added to `baseline.md`: Anthropic's
  published "loop engineering" framing (persistent instructions/skills/hooks driving an agent to
  iterate autonomously until a completion condition holds), which turns out to describe this
  repo's own weekly research routine, previously undocumented under that term. Bulk-evaluated 238
  previously-unitemized `claude-plugins-official` plugins (marketplace grew 295 → 310) — none
  recommended, same credential/no-mapped-workflow reasoning as prior passes; individually flagged
  `supabase` (redundant with Compass's existing platform connector) and a handful of others.
  Found and evaluated (not added) `AqueGen/model-routing`, a promising but too-new (3★, days old)
  third-party model-tier-routing plugin. No plugin or token-efficiency technique cleared the
  recommend bar this week — no `decisions/` entries drafted. See
  [`research/2026-09-19-prompting-techniques.md`](research/2026-09-19-prompting-techniques.md).

## 2026-09-12

- **Eighth weekly research pass: agent architectures & multi-agent patterns deep-dive
  (rotation's 2nd topic, revisited).** Found and added a new marketplace,
  `anthropics/claude-plugins-community` — a first-party Anthropic-run *community* marketplace
  (2,282 plugins, automated-security-scanned submissions) distinct from `claude-plugins-official`'s
  curated listing; logged an explicit distinction that an official host doesn't waive the
  per-plugin reputability bar, since the automated scan only checks for malicious code. No plugin
  recommended from it this week — too large to vet in one pass; future scans will use narrow
  topic-driven keyword searches against it instead of a full re-scan. Also evaluated and declined
  `davepoon/buildwithclaude` (3.4k★ discovery site, same infeasible-vetting-at-scale shape as the
  already-rejected `tonsofskills.com`) and six individual `claude-plugins-official` plugins
  (`atomic-agents`, `agent-sdk-dev`, `pydantic-ai`, `forge-skills`, `ai-plugins`, `cwc-makers`), all
  rejected on no-mapped-workflow or credential grounds. No plugin cleared the recommend bar. No new
  token-efficiency technique found (checked Anthropic's Messages-API context-editing/memory-tool
  features — real, but API-level for building custom agents, not applicable to how Claude Code
  itself delegates work). Deep-dive was a confirming pass: Anthropic's own published five-pattern
  coordination taxonomy (Orchestrator-Subagent, Generator-Verifier, Agent Teams, Message Bus, Shared
  State) maps cleanly onto three of `baseline.md`'s existing tiers, with Message Bus/Shared State
  noted as having no current analog or need. `baseline.md` updated in place for the marketplace
  addition, the taxonomy mapping, and refreshed plugin counts. See
  [`research/2026-09-12-agent-architectures-multi-agent-patterns.md`](research/2026-09-12-agent-architectures-multi-agent-patterns.md).

## 2026-09-05

- **Seventh weekly research pass: MCP servers & integrations deep-dive (rotation restarts).** No
  plugin cleared the recommend bar — `superself` (wshobson/agents, new this week) is the
  best-behaved memory/state-tracking candidate seen yet (zero auto-running hooks, unlike
  `pensyve`/`remember`/`claude-mem`) but was rejected on plain immaturity (~2-week-old repo, 4
  stars, solo/duo team); flagged as a revisit candidate. `claude-market/marketplace` evaluated as a
  new marketplace and not added (thin reputability signal, same shape as the already-rejected
  `xiaolai` marketplace). **Recommended (not applied): a `fork`-vs-fresh-subagent delegation rule
  for the token-efficiency section** — this harness's `Agent` tool supports a context-inheriting
  `subagent_type: "fork"` mode, previously undocumented here, that reuses the parent's prompt cache
  instead of restating context in a fresh subagent's prompt; recommended with an explicit
  boundary against using it for independent verification or when the parent context is mostly
  irrelevant bulk. See
  [`decisions/2026-09-05-token-efficiency-fork-subagent.md`](decisions/2026-09-05-token-efficiency-fork-subagent.md).
  `baseline.md` updated in place (not just flagged) for two smaller factual findings that don't
  need review-gating: the MCP section now documents three distinct MCP mechanisms instead of two
  (the account-level connector system, project `.mcp.json`, and this routine's own
  session/environment-level GitHub MCP tools), plus a native `SearchMcpRegistry` connector-directory
  search tool; the Agent architecture section now lists the `fork` subagent mode itself as an
  available capability (documenting that it exists is factual, separate from the token-efficiency
  policy recommendation about when to use it, which still goes through review). See
  [`research/2026-09-05-mcp-servers-integrations.md`](research/2026-09-05-mcp-servers-integrations.md).

## 2026-08-29

- **Sixth weekly research pass: tooling/infrastructure/automation deep-dive (rotation complete,
  wraps back to MCP servers & integrations next week).** No plugin cleared the recommend bar this
  week — `hol-guard` (wshobson/agents) came closest conceptually (local pre-install security
  vetting for AI agent tools, directly mirroring this repo's own mission) but was rejected on
  maturity/footprint grounds: v0.1.0, 4★, and it installs a fail-closed hook gating *every* tool
  call, a larger automatic-interception footprint than the memory tools (`pensyve`/`remember`/
  `claude-mem`) already rejected for smaller footprints with more track record. ~49 other
  previously-unitemized `claude-code-workflows` plugins evaluated and bulk-rejected (no mapped
  workflow, or duplicates of installed/bundled capability). `tonsofskills.com` identified as the
  same maintainer's project as the already-rejected `claude-code-plugins-plus-skills`, just
  rebranded — not a new marketplace. No new token-efficiency technique cleared the bar either; the
  research entry instead flags that three prior plugin recommendations and two token-efficiency
  drafts have been sitting unreviewed for 2-3 weeks. The substantive finding was for `baseline.md`
  itself: the bundled-skill roster had drifted from what was documented there, most notably
  `session-start-hook` — an unused, no-install skill that directly targets the "Sandbox bootstrap
  notes" pain point (fresh cloud sessions reinstalling tooling from scratch) this repo has
  documented as unresolved since its first day. See
  [`research/2026-08-29-tooling-infrastructure-automation.md`](research/2026-08-29-tooling-infrastructure-automation.md).

## 2026-08-22

- **Fifth weekly research pass: testing/verification deep-dive.** A confirming pass on both fronts
  that didn't clear the bar for a new install or a token-efficiency change — skills scan evaluated
  14 previously-unitemized plugins across both large marketplaces (rejected: credential
  requirements, no mapped workflow, or duplicates of installed/bundled capability — notably
  `avoid-ai-writing` duplicates the bundled `unslop` skill, and `javascript-typescript` duplicates
  the already-recommended `typescript-lsp`); token-efficiency check found nothing beyond
  platform-level mechanism already carved out of scope. The substantive finding was for
  `baseline.md` itself: added a dedicated "Testing / verification" section (previously just a line
  in Known gaps) documenting the native `/goal` command as an independent-evaluator verification
  mechanism in its own right, not just the reason `ralph-wiggum` was rejected — and flagged that the
  `Workflow` tool's adversarial-verify/judge-panel idioms are now on their second consecutive
  baseline mention as "documented, never actually used." `claude-code-workflows` marketplace shrank
  95 → 92 plugins (two previously-rejected plugins dropped out; not acted on). See
  [`research/2026-08-22-testing-verification-approaches.md`](research/2026-08-22-testing-verification-approaches.md).

## 2026-08-15

- **Weekly research pass: memory systems deep-dive + skills scan re-swept `claude-plugins-official`
  (284 → 286 plugins).** Recommended 2 new plugins for the next manual review to install (not
  installed in-session — cloud sandbox is ephemeral): `chrome-devtools-mcp` (live browser
  debugging/inspection, distinct from the already-installed `playwright`'s test-automation role) and
  `typescript-lsp` (first-party TypeScript language server, matches Compass's actual stack). ~13
  other newly-itemized plugins evaluated and rejected with reasons in `skills-seen.md`. See
  [`research/2026-08-15-memory-systems.md`](research/2026-08-15-memory-systems.md),
  [`decisions/2026-08-15-install-chrome-devtools-mcp.md`](decisions/2026-08-15-install-chrome-devtools-mcp.md),
  and
  [`decisions/2026-08-15-install-typescript-lsp.md`](decisions/2026-08-15-install-typescript-lsp.md).
- **Recommended a token-efficiency technique for `~/.claude/CLAUDE.md`**: use `.claude/rules/`
  path-scoped instruction files (loaded only when a matching file is read) instead of always-loaded
  `CLAUDE.md` content, for instructions that are genuinely conditional on file type/location — with
  an explicit boundary excluding anything that must hold regardless of which file is touched. Not
  yet applied — goes through manual review. See
  [`decisions/2026-08-15-token-efficiency-path-scoped-rules.md`](decisions/2026-08-15-token-efficiency-path-scoped-rules.md).
- **Updated `baseline.md`**: documented the subagent-level `memory` field and `.claude/rules/`
  (both new findings from this run), refreshed marketplace/plugin counts, noted the
  still-pending 2026-08-08 effort-routing recommendation.

## 2026-08-06

- **Installed `superpowers` plugin** (v6.2.0, user scope) — see
  [`decisions/2026-08-06-install-superpowers.md`](decisions/2026-08-06-install-superpowers.md).
- **Created this repo** — passive changelog/decision-log for how Claude's own setup evolves over
  time, across every project. See
  [`decisions/2026-08-06-toolkit-created.md`](decisions/2026-08-06-toolkit-created.md) and the full
  design at
  [`specs/2026-08-06-continuous-self-improvement-design.md`](specs/2026-08-06-continuous-self-improvement-design.md).
- **Set skills auto-install policy** (install new, review-before-retire old) — see
  [`decisions/2026-08-06-skills-autonomy.md`](decisions/2026-08-06-skills-autonomy.md).
- **Created and ran the weekly research routine for the first time.** Confirmed the routine's cloud
  sandbox is ephemeral, so its `claude plugin install` calls don't persist — corrected the skills
  lane to *recommend* rather than install in-session. See
  [`decisions/2026-08-06-fix-skills-autonomy-ephemeral-sandbox.md`](decisions/2026-08-06-fix-skills-autonomy-ephemeral-sandbox.md).
  First deep-dive (MCP servers & integrations) written manually from the same verified findings
  after the automated run hit a separate write-access blocker (see `baseline.md` known gaps) — see
  [`research/2026-08-06-mcp-servers-integrations.md`](research/2026-08-06-mcp-servers-integrations.md).
- **Evaluated 2 skills from a newly found marketplace** (`wshobson/agents`) — both declined
  (`pensyve`: unreviewed auto-running hooks; `tdd-workflows`: conflicts with superpowers). See
  `skills-seen.md`.
- **Second weekly research pass: agent architectures/multi-agent patterns + first full per-plugin
  skills scan of `claude-plugins-official` and `wshobson/agents`.** Recommended 3 plugins for the
  next manual review to install (not installed in-session — see the ephemeral-sandbox decision
  above): `playwright` (browser E2E, closes a gap already named in `baseline.md`), `block-no-verify`
  (enforces the existing no-bypass-flags git rule as a hook), and `claude-code-setup` (automates
  new-project tooling analysis). ~24 other plugins evaluated and rejected with reasons in
  `skills-seen.md`. Added `baseline.md`'s missing "Agent architecture / multi-agent tooling"
  section. See
  [`research/2026-08-06-agent-architectures-multi-agent-patterns.md`](research/2026-08-06-agent-architectures-multi-agent-patterns.md),
  [`decisions/2026-08-06-install-playwright.md`](decisions/2026-08-06-install-playwright.md),
  [`decisions/2026-08-06-install-block-no-verify.md`](decisions/2026-08-06-install-block-no-verify.md),
  and
  [`decisions/2026-08-06-install-claude-code-setup.md`](decisions/2026-08-06-install-claude-code-setup.md).
- **Write-access blocker resolved** — the GitHub App was granted access to all repos; the routine's
  third run (its second research pass) pushed successfully on the first try, confirmed live.
- **Installed the 3 recommended plugins** (`playwright`, `block-no-verify`, `claude-code-setup`) —
  manual review confirmed the routine's reasoning and installed all three in this persistent
  environment. See
  [`decisions/2026-08-06-manual-review-installs-three-recommended.md`](decisions/2026-08-06-manual-review-installs-three-recommended.md).
- **Broadened scope beyond skills** (user request) — the weekly routine now also does a
  token-efficiency check every run, not just a rotation topic, and the design's framing is now
  explicitly "state of the art AI-assisted software development," not skills-centric.
- **Created `~/.claude/CLAUDE.md`** — the missing mechanism for a *behavioral* decision (not just a
  plugin install) to actually change future sessions. First entries: explicit Haiku routing for
  cheap/high-volume subagent work, memory-system discipline. See
  [`decisions/2026-08-06-global-claude-md.md`](decisions/2026-08-06-global-claude-md.md) and
  [`research/2026-08-06-token-efficiency-and-agent-landscape.md`](research/2026-08-06-token-efficiency-and-agent-landscape.md).
- **Added an explicit quality-first boundary to the token-efficiency section** — user feedback:
  don't trade quality for token savings. Every efficiency instruction now has to state when it
  doesn't apply, not just when it does; enforced as a standing requirement for future entries too,
  not just today's two. See
  [`decisions/2026-08-06-quality-first-guardrail.md`](decisions/2026-08-06-quality-first-guardrail.md).
- **Design/UI-UX deep-dive pass (user request), installed 3 plugins.** Researched SOTA UI/UX and
  product design practices for AI-assisted development; grounded the "generic AI aesthetics"
  finding against Compass's own unstyled `create-next-app` scaffold. Installed
  `frontend-design@claude-plugins-official` (official, counters generic-AI-aesthetics), and
  `modern-web-guidance@claude-plugins-official` (Google Chrome, 130+ web-platform/accessibility
  checks) directly. Added a new marketplace (`impeccable`, github.com/pbakaus/impeccable, 56k★) and
  installed its single companion plugin after reading its auto-running hook script to confirm it's
  purely local/no-network before install. Evaluated and declined Figma/design-handoff MCP (no
  current use case). Added a "UI/UX & product design" standing-instructions section to
  `~/.claude/CLAUDE.md` (accessibility floor, screenshot self-critique loop, heuristic review pass)
  so the installs actually get used, not just sit there. See
  [`research/2026-08-06-design-ux.md`](research/2026-08-06-design-ux.md),
  [`decisions/2026-08-06-install-design-plugins.md`](decisions/2026-08-06-install-design-plugins.md),
  and
  [`decisions/2026-08-06-design-workflow-standing-instructions.md`](decisions/2026-08-06-design-workflow-standing-instructions.md).
- **Sharpened the Haiku-routing boundary** — user feedback while reviewing the Compass redesign's
  Phase 0 plan: the rule should read as covering genuinely repetitive/simple tasks only, explicitly
  not complete redesigns or critical implementations, and a *product's* own AI-feature model choice
  (e.g. an Edge Function calling the Anthropic API) is a separate decision from *my own* subagent
  dispatch, not something to justify by analogy to this rule. See
  [`decisions/2026-08-06-sharpen-haiku-routing-boundary.md`](decisions/2026-08-06-sharpen-haiku-routing-boundary.md).

## 2026-08-08

- **Third weekly research pass: prompting techniques deep-dive + full re-scan of both large
  marketplaces.** Mostly a confirming pass — current practice (superpowers' plan/execute/verify
  skills, this repo's own `specs/` workflow) already tracks 2026 SOTA (spec-driven development,
  meta-prompting/step-back decomposition); closes the "no prompting-technique review done" known
  gap in `baseline.md`. Recommended installing `context7` (Upstash library-docs MCP server,
  348k+ installs, no required credential) and adding explicit effort-level routing for
  `Workflow`-dispatched agents to `~/.claude/CLAUDE.md`'s token-efficiency section (a second,
  independent knob alongside the existing Haiku model-routing rule, same required
  never-for-judgment-work boundary). Evaluated and rejected `claude-mem` (88.5k★ memory plugin) —
  reads as the strongest cross-project-memory candidate yet by reputation, but its actual
  `hooks.json` shows a *larger* automatic background-worker footprint than the two previously
  rejected memory tools, not a smaller one; flagged as a revisit candidate contingent on an actual
  source read, not ruled out permanently. ~15 other plugins evaluated individually and rejected
  (credential requirements, no mapped workflow, overlap with installed skills); the long unlisted
  tail in both large marketplaces (~327 plugins combined) bulk-logged with reasoning rather than
  itemized. See
  [`research/2026-08-08-prompting-techniques.md`](research/2026-08-08-prompting-techniques.md),
  [`decisions/2026-08-08-install-context7.md`](decisions/2026-08-08-install-context7.md), and
  [`decisions/2026-08-08-token-efficiency-effort-routing.md`](decisions/2026-08-08-token-efficiency-effort-routing.md).
