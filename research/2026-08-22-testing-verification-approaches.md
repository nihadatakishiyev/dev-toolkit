# Weekly research — 2026-08-22

**Deep-dive topic:** Testing/verification approaches (5th in rotation, after MCP servers &
integrations → agent architectures & multi-agent patterns → prompting techniques → memory systems)

## Skills scan

Ephemeral sandbox (per `baseline.md`) — re-added all five previously-configured marketplaces to
inspect current listings: `claude-plugins-official`, `superpowers-dev` (obra/superpowers),
`claude-code-workflows` (wshobson/agents), `impeccable` (pbakaus/impeccable), `thedotmack`
(thedotmack/claude-mem).

**Marketplace-level changes:**
- `claude-plugins-official`: **286 plugins, unchanged** from the 2026-08-15 count. No new plugins
  to itemize from a count-diff standpoint.
- `claude-code-workflows` (wshobson/agents): **92 plugins, down from 95** (2026-08-08/08-15
  count). `qa-orchestra` and `ciagent` (both individually rejected 2026-08-06) are no longer
  present; a third plugin also dropped out of the listing but wasn't identified by name (not
  itemized before, so nothing to note losing). Not acted on — this routine doesn't track removals
  from third-party marketplaces, only additions (per the design's asymmetric treatment of
  "recommend new / never auto-remove existing").
- `superpowers-dev`, `impeccable`, `thedotmack`: unchanged (1 plugin each).
- Web search for other reputable marketplaces not yet configured found no new *installable*
  (`claude plugin marketplace add`-compatible) source. One notable discovery: **qaskills.sh**
  (PramodDutta/qaskills, 450+ testing-focused skills for 30+ agents including Claude Code) — but
  it's a different distribution mechanism entirely, not a `.claude-plugin/marketplace.json`
  marketplace. It installs individual `SKILL.md` files into `~/.claude/commands` via its own `npx
  @qaskills/cli add <skill-name>` CLI, bypassing `claude plugin` altogether. Logged here as a
  discovery, same treatment as the `claudemarketplaces.com`/`aitmpl.com` directory sites already on
  record — a place to browse from, not something this routine can `marketplace add`. Individual
  skills from it could be manually vetted and copied in during a manual review pass if a specific
  testing need arises, but that's a different, higher-touch action than what this scan does.

**New plugins evaluated this run** (via testing/verification-focused keyword search across both
large marketplaces, prompted by this week's deep-dive topic — not an exhaustive re-scan of all
92+286 entries, consistent with prior weeks' bulk-rejection precedent for the long unitemized
tail):

From `claude-plugins-official` (previously-unitemized, all part of last week's ~262-strong bulk
bucket, individually named for the first time this run):
- `42crunch-api-security-testing` — rejected, requires a 42Crunch account/API key.
- `circle-skills` — rejected, stablecoin/crypto payments (Circle/USDC) — no mapped project.
- `circleback` — rejected, requires a Circleback account for meeting/calendar integration.
- `teamcity-cli` — rejected, requires a TeamCity CI server, not in use anywhere.

From `claude-code-workflows` (wshobson/agents, previously-unitemized):
- `performance-testing-review` — rejected. "Performance analysis, test coverage review, and
  AI-powered code quality assessment" — overlaps bundled `review`/`security-review` +
  superpowers' `verification-before-completion`; no active performance-regression-testing
  workflow to attach it to, same reasoning as the earlier `codspeed` rejection.
- `deployment-validation` — rejected. "Pre-deployment checks, configuration validation, and
  deployment readiness assessment" — no active deployment pipeline this routine tracks.
- `data-validation-suite` — rejected. "Schema validation, data quality monitoring, streaming
  validation pipelines, input validation for backend APIs" — no mapped current project need;
  worth revisiting if a project builds out a real API-validation surface.
- `javascript-typescript` — rejected. Generic JS/TS language-bundle skill — overlaps the
  already-recommended `typescript-lsp` plus Claude's own built-in language capability, same
  precedent as prior per-language bundle rejections (2026-08-15's `pyright-lsp`/etc. batch).
- `llm-application-dev` — rejected. LangGraph/RAG/vector-search specific — no project here uses
  LangGraph or a RAG pipeline.
- `avoid-ai-writing` — rejected. Direct duplicate of the platform-bundled `unslop` skill (already
  available, no install needed) — same "duplicate of bundled skill" precedent as the earlier
  `skill-creator` (plugin) rejection.
- `skill-forge-essentials` — rejected. "Behavioral skills for AI-specific failure patterns"
  (context-compaction amnesia, hidden code debt) — vague, overlaps superpowers'
  `verification-before-completion`/`systematic-debugging` without a clearly distinct mechanism.
- `api-scaffolding` — rejected. No active API-scaffolding workflow.
- `distributed-debugging`, `error-diagnostics` — rejected. Both overlap `debugging-toolkit`
  (already rejected 2026-08-08) and superpowers' `systematic-debugging`.

**Net result: nothing cleared the recommend bar this week.** Every candidate either required a
new credential, had no mapped current-project workflow, or duplicated something already
installed/bundled. See `skills-seen.md` for the full dated log.

## Token-efficiency / context-management check

Researched current techniques: Anthropic's own context-editing + memory-tool combination (per
Anthropic's own published numbers, ~29% improvement from context editing alone, ~39% combined with
the memory tool, and up to 84% token reduction in long-horizon/100-turn evaluations). This is a
genuinely notable number, but it's a **Claude API/platform-level mechanism** (automatic clearing of
stale tool calls/results as context fills) — the same category `baseline.md` and the spec already
carve out as *not* what this weekly check is about ("most of the actual mechanism here is
platform-managed... this check is about workflow-level technique"). Nothing here is a knob a
`~/.claude/CLAUDE.md` instruction would turn.

Beyond that: model-tier routing (Opus/Sonnet/Haiku by task) and "don't dump everything into
context" guidance both came up in general 2026 best-practice roundups, but both are already
reflected in the existing `~/.claude/CLAUDE.md` token-efficiency section (Haiku routing for
cheap/high-volume subagent work, preferring targeted reads over whole-file reads, and the
2026-08-08 effort-level routing addition for `Workflow` dispatch).

**No new workflow-level technique found this week that isn't already reflected.** Confirming pass,
not a gap-closing one — no `decisions/` entry written, consistent with the spec's "if nothing
substantive turned up, say so" fallback.

## Deep-dive: Testing/verification approaches

Compared against `baseline.md`, which currently documents: superpowers'
`test-driven-development`/`verification-before-completion`/`systematic-debugging` skills
(installed, "no established testing/verification skill usage pattern... unused so far" as of the
last note), and the `Workflow` tool's documented adversarial-verify/judge-panel idioms
("available... not yet used on any real task").

**Finding 1 — the native `/goal` command is a real verification mechanism baseline doesn't credit
as one.** `baseline.md` currently mentions `/goal` only in passing, as the reason `ralph-wiggum`
was rejected ("superseded by Claude Code's own built-in `/goal` command"). Digging into what it
actually does (shipped Claude Code v2.1.139, May 2026): you declare a completion condition, and
after every turn a **separate, fresh evaluator model** (defaults to Haiku) checks that condition
against the conversation so far and returns yes/no + a short reason — a "no" feeds back as guidance
for the next turn. This is architecturally the same *independent-evaluator* idea the 2026
LLM-as-judge literature converges on (the model doing the work should not be the one grading its
own completion) — it's just already built into the harness, no plugin or `Workflow` script needed.
Its limitation: the evaluator only sees what's already in the conversation, it doesn't independently
re-run commands or inspect files itself, so it's a check on *stated* completion, not a substitute
for an actual test run. Distinct from `Stop` hooks (which enforce settings/team-scoped, not
session-scoped, and *can* shell out to run real checks) — the two are complementary, not
competing.

**Finding 2 — 2026 LLM-as-judge best practice is basically a formalization of the "adversarial
verify" idiom the `Workflow` tool already documents**, not a new pattern to add: prefer clear
pass/fail rubrics over vague numeric scales, treat the judge as versioned/tested infrastructure
rather than a throwaway prompt, and use a multi-persona panel (not a single verifier) for anything
domain-complex — all of which the `Workflow` tool's built-in adversarial-verify and judge-panel
idioms (documented in `research/2026-08-06-agent-architectures-multi-agent-patterns.md`) already
implement structurally. One more specific technique surfaced worth naming: the **"tool receipt"
pattern** — grounding a claimed action to an actually-logged tool-call execution (if the agent says
it searched for something but the transcript shows no search call, that's a concrete, checkable
signal of a false completion claim) — conceptually the same discipline as superpowers'
`verification-before-completion` skill already requiring build output/test results/screenshots as
evidence, not just a "looks done" self-report.

**Overall read:** this is, again, mostly a *confirming* pass rather than a gap-closing one — the
skills/tools already in place (superpowers TDD/verification skills, `Workflow` adversarial-verify)
are conceptually aligned with 2026 SOTA. The concrete, actionable gap is usage, not tooling: the
`Workflow` tool's verify/judge-panel patterns are now on their **second** consecutive baseline
mention as "documented, zero real-task usage" — worth flagging explicitly rather than letting that
line just get copy-pasted forward again next rotation. `/goal` specifically is undocumented as a
verification mechanism in its own right and costs nothing to start using (no install, no plugin) —
the clearest "should actually pick this up" item this run.

## Candidate improvements (not yet acted on)

| Candidate | Impact | Effort | Notes |
|---|---|---|---|
| Document `/goal` in `baseline.md` as a testing/verification mechanism (not just a ralph-wiggum replacement), and actually use it on a real long-running/autonomous task to see how the evaluator behaves in practice | M | L | No install needed, native to the harness. Addressed partially this run — see baseline.md update below — but "actually use it once" is still open. |
| Give the `Workflow` tool's adversarial-verify/judge-panel idioms a real first run (e.g. on a genuinely ambiguous code-review or design-decision task) instead of letting "documented but unused" persist a third time | M | M | Needs a concrete task to attach it to, not speculative use — same reasoning baseline already applies to itself. |
| Investigate `qaskills.sh`'s individual skills (not the whole directory) if a project develops a concrete Playwright/Cypress/perf-testing need beyond what's already installed | L | M | Different distribution mechanism (its own CLI, not `claude plugin`) — would need per-skill trust vetting, not a blanket add. |
| Revisit `data-validation-suite` (wshobson) if a project builds a real API-input-validation surface | L | L | Currently no mapped workflow; noted for revisit only. |
