# Weekly research pass — 2026-10-10

Deep-dive topic: **tooling/infrastructure/automation** (rotation's 6th topic, revisiting the
2026-08-29 pass — the rotation has now completed one full lap:
MCP servers → agent architectures → prompting → memory → testing/verification → tooling/
infrastructure, and back to MCP servers next week).

## 1. Skills scan

Re-added all six previously-configured marketplaces (ephemeral sandbox — see baseline.md):
`claude-plugins-official`, `superpowers-dev`, `claude-code-workflows`, `impeccable`, `thedotmack`,
`claude-community`.

Counts vs. last week (2026-10-03):
- `claude-plugins-official`: **315 → 315 (unchanged)**.
- `claude-code-workflows`: **94 → 94 (unchanged)**.
- `impeccable`, `superpowers-dev`, `thedotmack`: unchanged (1/1/2 plugins).
- `claude-community`: **2283 → 2284** (+1).

With the two largest first-party marketplaces showing zero net growth, there's nothing new to
individually evaluate there this week — every name in both has already been itemized or
bulk-rejected across the last several passes (see `skills-seen.md`). `claude-community`'s +1 is too
marginal to chase down by name against its 2,284-entry listing (no per-entry timestamp field exists
to isolate it, and the established precedent for this marketplace — set 2026-09-12, reconfirmed
every week since — is narrow keyword searches, not full re-diffs, given its scale). Ran a
tooling/automation-focused keyword sweep (`automation`, `infrastructure`, `ci-cd`, `pipeline`,
`devops`, `deploy`, `observability`, `orchestrat`, `cron`, `scheduler`, `terraform`, `iac`,
`gitops`) against it anyway: 429 name/description matches, confirming (again) the same shape
already on record — a long tail of single-maintainer SaaS-integration and framework-specific
plugins, none with an independent reputability signal beyond this marketplace's own automated
security scan. No plugin recommended from it, consistent with every prior pass.

Web search for new marketplaces not yet configured found nothing beyond the discovery-directory
sites already on record (`claudemarketplaces.com`, `aitmpl.com`) and search noise from unrelated
third-party blog aggregators. No new marketplace added this week.

**Verdict: no new skill or marketplace cleared (or failed) evaluation this week — genuinely quiet
scan**, not a skipped one. No `skills-seen.md` plugin rows added; one scan-confirmation row added
for the record (see that file).

## 2. Token-efficiency / context-management check

Found a concrete, actionable gap this week: **Claude Haiku 5.5** (shipped with Claude Code
v2.1.293, 2026-10-07; now the default Haiku model on the Anthropic API) carries a **5x per-token
price multiplier past 100K tokens of prompt**, on top of being natively 1M-context and the first
Haiku model with its own effort control. The existing Haiku-routing bullet in
`~/.claude/CLAUDE.md`'s token-efficiency section assumes routing to Haiku is a straightforward cost
win for the cheap/high-volume work it targets — true for small prompts, silently false for a
large one (e.g. a `fork`-mode dispatch carrying a long inherited conversation, or a task reading
several full files). Drafted `decisions/2026-10-10-token-efficiency-haiku-pricing-cliff.md`: a
narrow clause on the *existing* bullet (not a new one) — keep Haiku-routed prompts small, and
route elsewhere instead of trimming content a task actually needs, with the required boundary that
this is a cost-routing refinement only and never overrides the existing never-for-judgment-work
rule. Quality cost: zero — it only changes which model/how much prompt a cost-motivated dispatch
uses.

Separately, found that the plain `Agent` tool itself (not just the `Workflow` tool's `agent()`
calls) **now exposes an `effort` parameter** (Claude Code v2.1.292, 2026-10-06) — directly
contradicting a factual claim in `decisions/2026-08-08-token-efficiency-effort-routing.md` ("the
plain `Agent` tool has no `effort` parameter to set"), which is now stale. Checked whether this
opens a new routing opportunity the way the 2026-08-08 Workflow-effort finding did: it doesn't. The
tool's own built-in description already restricts the parameter to "only when the user, or
instructions such as CLAUDE.md or a skill, explicitly ask... never on your own judgment" — a
stricter, harness-enforced boundary than anything a CLAUDE.md bullet could add on top. Nothing to
recommend here beyond the baseline correction (below) — the native guardrail is already exactly
right, autonomous effort-routing for the plain `Agent` tool is correctly *not* offered.

Also noted, not actionable: the Anthropic API's `clear_tool_uses_20250919` / `clear_thinking_20251015`
context-editing strategies (server-side tool-result/thinking-block trimming past a configurable
threshold) are a genuine token-efficiency mechanism, but per this routine's own design (see
`specs/2026-08-06-continuous-self-improvement-design.md`), platform-managed mechanisms like this
are explicitly out of scope for this check — the check is about workflow-level technique (how work
gets delegated), not re-litigating platform internals every week. Recorded here for completeness
only.

## 3. Deep-dive: tooling/infrastructure/automation

Three genuinely new mechanisms found, all from the same burst of Claude Code releases this week
(v2.1.287–v2.1.294, 2026-10-01 through 2026-10-08):

- **Claude Mods** (v2.1.287, 2026-10-01, on by default). A new plugin type distinct from skills,
  hooks, and MCP servers: JS/TS handler functions that run *inside* the Claude Code process itself
  and can observe, rewrite, or replace internal events (a tool call, a prompt being sent) or alter
  the UI (panels, banners, redrawn elements). Unlike external shell hooks, mods run with the
  installing user's own permissions and are not sandboxed — a materially different trust model from
  everything else in baseline.md's extensibility inventory (skills, `.claude/rules/`, hooks, MCP).
  This account's own available-skills listing already includes a bundled `plugin-authoring` skill
  for building/debugging mods, confirming this is live in this account, not a future feature. No
  concrete driver yet for building one for any tracked project (same "not currently actionable"
  shape as several other baseline entries) — noted as a new mechanism to be aware of, not a gap to
  close.
- **Native subagent-effort argument on the `Agent` tool** (v2.1.292, 2026-10-06) — covered above
  under token-efficiency, listed here too since it's also a tooling/automation-mechanism change
  (the dispatch surface itself gained a new parameter), not just a cost lever.
- **`--marketplace` flag on `claude plugin install`** (v2.1.292) — lets an install specify which
  marketplace to pull a same-named plugin from directly, instead of relying on marketplace-add
  order to disambiguate. Minor; doesn't change this routine's own evaluation process, worth knowing
  if a future plugin name collision needs resolving.
- **"You Should Know"** (v2.1.287, opt-in, off by default) — a side agent that watches a session
  and flags things the user or Claude might have missed. Interesting in principle (this repo's own
  mission is catching things that'd otherwise be missed), but it requires enabling telemetry
  (`/plugin enable cc-plugin-you-should-know@builtin` on a telemetry-on session) — a scope/privacy
  tradeoff this routine isn't positioned to decide unilaterally. Logged as a research candidate
  below, not drafted as a `decisions/` entry (it's a built-in feature flag, not a marketplace
  plugin, so it falls outside the skills-scan carve-out's specific mechanism — but the same
  "manual review decides" spirit applies).

Compared against baseline.md: the "Agent architecture" and "Global config" sections are both now
stale on specific points (see baseline update below). No existing tooling/automation gap from the
2026-08-29 pass was closed this week — that pass's main finding (`session-start-hook` for sandbox
bootstrap, still "candidate fix, not yet applied") remains exactly where it was, not revisited this
week since nothing new touches it.

## Candidate improvements (not yet acted on)

| Candidate | Impact | Effort | Status |
|---|---|---|---|
| Haiku-routing pricing-cliff boundary (100K-token cost cliff) | M | L | Decision drafted this run (`decisions/2026-10-10-token-efficiency-haiku-pricing-cliff.md`), pending manual-review application to `~/.claude/CLAUDE.md` |
| Correct stale "`Agent` tool has no `effort` param" claim | L | L | Baseline corrected this run (see below); no CLAUDE.md action needed, native guardrail already right |
| Claude Mods — build one for this repo or Compass | M (if a use case appears) | M | No concrete driver yet; revisit opportunistically during manual review, same as `session-start-hook` |
| "You Should Know" opt-in agent | M | L | Requires enabling telemetry — flag for a human decision during manual review, don't enable unilaterally |
| `session-start-hook` for this repo's own sandbox bootstrap (carried over from 2026-08-29) | M | L | Still not applied — still a candidate, not newly found this week |
| `Workflow` tool's Generator-Verifier idiom, still zero real-task usage (5th consecutive mention) | M | M | Carried over again — genuinely overdue for a forced try, not a new finding |
