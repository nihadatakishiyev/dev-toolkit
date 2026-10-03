# Weekly research — 2026-10-03

**Deep-dive topic:** Testing/verification approaches (5th in rotation, revisited — after MCP
servers & integrations → agent architectures & multi-agent patterns → prompting techniques →
memory systems, rotation wraps to tooling/infrastructure/automation next week)

## Skills scan

Ephemeral sandbox (per `baseline.md`) — re-added all six previously-configured marketplaces to
inspect current listings: `claude-plugins-official`, `superpowers-dev` (obra/superpowers),
`claude-code-workflows` (wshobson/agents), `impeccable` (pbakaus/impeccable), `thedotmack`
(thedotmack/claude-mem), `claude-community` (anthropics/claude-plugins-community).

**Marketplace-level changes:**
- `claude-plugins-official`: **315 plugins, up from 314.** True week-over-week growth is just +1,
  but diffing the full current name list against everything previously recorded in this file (not
  just last week's named additions) turned up 212 names that had never been individually itemized
  by name before — i.e. the long-running "unlisted bulk tail" from the 2026-08-08/2026-08-29/
  2026-09-19 entries is still mostly un-spot-named, not that 212 plugins actually appeared this
  week. Chipped away at that tail the same incremental way those entries did: 8 new names
  individually evaluated below, the remaining ~204 bulk-rejected under the same established
  categories. Did not try to pin down which single name is the actual +1 — not worth the effort at
  this scale.
- `claude-code-workflows`: **94 plugins, unchanged.**
- `superpowers-dev`, `impeccable`: unchanged (1 plugin each). `thedotmack`: unchanged (2 plugins).
- `claude-community`: **2,283 plugins, up from 2,282** (second pass where this one has grown at
  all). Still far too large to vet in one weekly pass per the 2026-09-12 policy — did a narrow
  testing/verification-keyword search (`test`, `verif`, `qa`, `eval`, `coverage`, `assert`) instead
  of a full scan, per that same policy. Surfaced a few plausible-sounding hits (`meticulous` —
  visual-regression-testing skills; `agentiqa`/`agentic-beta-testers`/`android-emulator-qa-plugin` —
  agentic app-QA tools) but none have an independent reputability signal beyond this marketplace's
  own automated security scan (no visible star count, maintainer track record, or usage data
  surfaced for any of them) — same bar that has kept everything from this marketplace unrecommended
  so far. Not evaluated further; noted as discovery only.
- Web search for marketplaces not yet configured found one genuinely new, concretely installable
  one and two not worth adding:
  - **`ComposioHQ/awesome-claude-plugins`** (1.9k★/610 forks, backed by Composio, a real
    tool-integration company) — added this session to inspect its listing (24 plugins). Every
    single one either duplicates capability already installed/bundled here (`frontend-design`,
    `security-guidance`, `agent-sdk-dev`, design/git/PR/code-review/debugging tooling already
    covered by `impeccable`/superpowers/bundled skills) or fails the no-credential/mapped-workflow
    bar (`connect-apps` needs a Composio account per connected app; `mcp-builder`/`perf`/
    `documentation-generator`/`backend-architect`/`changelog-generator`/`developer-growth-analysis`
    have no concrete mapped workflow here). Same "strict subset, nothing unique" verdict as the
    2026-08-06 `anthropics/claude-code` marketplace — reputable host, but not kept as a standing
    marketplace since there's nothing here it uniquely unlocks. Full per-plugin reasoning in
    `skills-seen.md`.
  - `alirezarezvani/claude-skills` — 27.3k★, by far the highest star count of any discovery-site
    candidate evaluated to date, but 380+ skills from a single maintainer with no independent
    per-entry vetting, built for cross-tool portability (Codex/Cursor/Gemini CLI/8 more) rather
    than Claude Code specifically. Same infeasible-vetting-at-scale shape as the already-rejected
    `davepoon/buildwithclaude` (3.4k★) — the established precedent that a higher star count doesn't
    change the scale problem applies even more strongly here. Not added.
  - `Sagargupta16/claude-skills` — 5 stars, single maintainer, skills concentrated on the FARM
    stack (FastAPI/React/MongoDB) — no tracked project uses that stack, and the star count alone
    gives no independent reputability signal, same bar as `xiaolai`/`dashed`. Not added.

**New plugins evaluated this run:**

From `claude-plugins-official` (previously-unitemized bulk-tail names, picked as representative
rather than exhaustive):
- `lua-lsp` — rejected, same unused-language LSP-bundle precedent as the 2026-08-15 sweep.
- `redis-development` — rejected, no tracked project uses Redis.
- `prisma` — rejected, no tracked project uses Prisma (Compass's data layer is the Supabase
  platform connector, not a standalone ORM).
- `cloudflare`, `firebase`, `mongodb-atlas` — rejected, each a managed-hosting/BaaS platform
  needing its own account; no tracked project is hosted on any of the three.
- `clickhouse`, `clickhouse-best-practices` — rejected, no tracked project runs a data-analytics
  workflow, same bar as `duckdb-skills`/`bigquery-data-analytics`.
- `qt-development-skills` — rejected, unused desktop-GUI framework.
- `math-olympiad`, `math-proof` — rejected, unrelated domain, no mapped workflow category.
- `unreal-engine-skills-for-claude-code` — rejected, no game-development project tracked here.

From `ComposioHQ/awesome-claude-plugins` (all 24 plugins individually evaluated, see
`skills-seen.md` for the full breakdown) — none recommended.

**Net result: nothing cleared the recommend bar this week.** Every candidate either required a new
credential, had no mapped current-project workflow, or duplicated something already
installed/bundled. See `skills-seen.md` for the full dated log.

## Token-efficiency / context-management check

Checked for anything new beyond what `baseline.md` already documents (prompt caching, harness-level
auto-compaction, Haiku model-routing, `Workflow` effort-level routing, the `fork` subagent mode, and
`defer_loading`/Tool Search for large tool catalogs). Nothing new surfaced this week:

- Context compaction and the Messages-API context-editing/memory-tool combination both came up
  again in general roundups — same platform-level mechanism already carved out of scope (per the
  spec: "this check is about workflow-level technique," not re-litigating the platform's own
  internals).
- Deferred/lazy tool-schema loading (the mechanism this very session's own tool list demonstrates —
  `ListMcpResourcesTool`, `Monitor`, several `mcp__github__*` tools all arrived as deferred stubs
  this run) was reconfirmed working as already documented (found 2026-08-08) — nothing changed
  about how to use it.
- No new delegation/compression/routing pattern found that isn't already reflected.

**No new workflow-level technique found this week that clears the quality bar.** Confirming pass —
no `decisions/` entry written.

## Deep-dive: Testing/verification approaches

Compared against `baseline.md`'s existing "Testing / verification" section: superpowers' TDD/
verification skills (installed, still unexercised on a real task), the `Workflow` tool's
adversarial-verify/judge-panel idioms (documented, still unexercised — now flagged three times
running as of the last pass), and the native `/goal` command (independent-evaluator mechanism,
Haiku-backed, limited to judging what's already in the conversation).

**Finding 1 — `claude plugin eval` is a real, already-live verification mechanism not yet in
`baseline.md` at all**, and it's directly relevant to *this routine's own* skills-scan job, not just
a generic testing feature. Shipped in Claude Code v2.1.269 (Sept 11 2026); this session is running
v2.1.288, so it's live here now (`claude plugin --help` lists `eval` directly, and the
`claude-code-guide` subagent's own description already name-drops it). How it works: cases live
under a plugin's `evals/` directory (prompt + one or more graders); six grader types ship — four
free and computed straight from the transcript/filesystem (`regex`, `tool_used`, `tool_order`,
`file_exists`) plus two billable ones (`llm` judge, `baseline` judge); by default each case runs
multiple times with the plugin loaded and multiple times without, and the gap between the two arms
is the score; `claude plugin eval init` can scaffold cases/graders for a plugin that doesn't have
any yet; CI-oriented flags (`--threshold`, `--max-cost-usd`) let a build gate on the score. Found
three plugins in currently-configured marketplaces that already ship an `evals/` directory
(`claude-code-workflows` has one at its root; `thedotmack`'s `mode-creator` skill; `claude-plugins-
official`'s own `math-olympiad` skill) — confirms this isn't just a documentation claim, plugins are
actually shipping these. Did not actually invoke `claude plugin eval` against any of them this run
(each invocation runs real, billable Claude Code sessions per the with/without-arm design — not a
free local check, and confirming the documented mechanism works didn't need spending real API calls
to prove it firsthand for a routine built to stay low-effort).

**Why this matters beyond "a feature exists":** every week this routine evaluates plugin candidates
by reading descriptions, stars, and hook scripts — judgment calls about reputation and fit, not
measured behavior. For a plugin that ships (or could be made to ship, via `eval init`) its own eval
suite, `claude plugin eval` gives an actual pass/fail/score signal instead of pure reputation-based
guessing — exactly the kind of independent-verification-over-self-report discipline the
`Workflow`-tool's adversarial-verify idiom already preaches, just applied to *this routine's own*
recommend/reject calls rather than to code changes. Most concretely useful for the handful of
plugins currently sitting in "immature, revisit later" limbo (`superself`, `AqueGen/model-routing`,
`hol-guard`) — if any of them ship or gain an eval suite before their next revisit, running it would
turn "4★, 2 weeks old, revisit when it matures" into an actual measured signal instead of a waiting
game based purely on star count and repo age.

**Finding 2 — the "verification is getting harder, not easier" framing is worth naming, even though
it doesn't change any current practice.** 2026 research framing (e.g. "The Verification Horizon: No
Silver Bullet for Coding Agent Rewards") argues that as generator models get stronger, reliably
verifying their output is becoming the harder half of the problem, not the easier one — stronger
policies increasingly find ways to satisfy a check's letter without its spirit (retrieving solution
artifacts, tampering with tests) the same way a human might game a rubric. This reframes *why* the
`Workflow` tool's adversarial-verify/judge-panel idiom and superpowers'
`verification-before-completion` (requiring actual build output/test results/screenshots as
evidence, not a self-report of being done) matter — not just nice-to-have discipline, but the
correct response to a problem that's structurally getting worse as models improve. No action
item — it's a conceptual reinforcement of an already-identified gap, not a new tool or technique.

**Overall read:** another confirming pass on tooling, but the `claude plugin eval` finding is a
genuinely new, actionable mechanism — unlike `/goal` (2026-08-22) it's not just "undocumented here,"
it had zero prior mention in this repo at all. The `Workflow` tool's adversarial-verify/judge-panel
idioms are now on their **fourth** consecutive mention as "documented, zero real-task usage" since
2026-08-22 (reinforced 2026-09-12, 2026-09-19, and again by the Verification Horizon framing this
week) — baseline already called this "genuinely due for a forced try," and that's now one rotation
cycle (six weeks) overdue without a concrete trigger. Worth the manual-review pass treating this as
a standing item to actively watch for an opportunity to use, not just re-flag again next time this
topic comes around.

## Candidate improvements (not yet acted on)

| Candidate | Impact | Effort | Notes |
|---|---|---|---|
| Document `claude plugin eval` in `baseline.md`'s Testing/verification section as a native mechanism, and consider using it against any future "immature, revisit later" plugin candidate that ships an `evals/` dir, instead of waiting on star count/repo age alone | M | L | No install needed, native to the harness (confirmed live in this session's Claude Code version). Addressed partially this run — see baseline.md update below. |
| Give the `Workflow` tool's adversarial-verify/judge-panel idioms a real first run on a genuinely ambiguous task | M | M | Fourth consecutive mention as unused; needs a concrete task to attach it to, not speculative use. Flagged as overdue, not just re-noted. |
| Spot-check whether `superself` or `AqueGen/model-routing` (both on the "revisit when mature" list) have grown a maintained `evals/` suite next time either comes up for revisit | L | L | Would let a revisit use `claude plugin eval`'s actual score instead of just re-checking star count/repo age. |
| Finish individually itemizing `claude-plugins-official`'s long-standing unlisted bulk tail (now ~204 names still un-spot-named after this week's 8) | L | M | Pure bookkeeping — no plugin found in it has cleared the recommend bar in any pass so far; low expected value, but the tail doesn't shrink on its own. |
