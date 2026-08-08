# Weekly research — 2026-08-08

**Deep-dive topic:** Prompting techniques (3rd in rotation, after MCP servers & integrations →
agent architectures & multi-agent patterns)

## Skills scan

Sandbox is ephemeral as documented in `baseline.md` — re-added all four previously-known
marketplaces (`claude-plugins-official`, `superpowers-dev`, `claude-code-workflows`, `impeccable`)
to inspect their current listings. Both large marketplaces grew slightly since the last full scan:
`claude-plugins-official` 278 → 284 plugins, `claude-code-workflows` (`wshobson/agents`) ~94 → 95.

**New marketplace added:** `thedotmack` (github.com/thedotmack/claude-mem) — 88.5k★, very actively
maintained, single-plugin marketplace. Added for future-scan visibility even though its plugin is
rejected this run (see below).

**New marketplace found, not added:** `jeremylongshore/claude-code-plugins-plus-skills` — 2.6k★,
but a single-maintainer 471-plugin/3,069-skill "kitchen sink" marketplace with no independent
vetting signal beyond the star count; the scale itself makes per-plugin review infeasible for this
routine. Also reconfirmed `claudemarketplaces.com` and newly noticed `aitmpl.com` as curated
*directory* sites (useful discovery sources, not installable marketplaces themselves).

**New plugin recommended:** `context7` (Upstash, `claude-plugins-official`) — library documentation
lookup MCP server, works anonymously, 348k+ installs, directly useful on essentially every coding
session. See `decisions/2026-08-08-install-context7.md`.

**Notable rejection — `claude-mem` (thedotmack):** the strongest candidate yet for the long-standing
"no cross-project memory system" gap by reputation (88.5k★, far more established than the
previously-rejected `pensyve`/`remember`), but reading its actual `hooks.json` found *six*
auto-running hooks (`Setup`, `SessionStart`×2, `UserPromptSubmit`, `PostToolUse` on every tool,
`PreToolUse` on every `Read`, `Stop`) driving a persistent background worker service with full shell
access — a larger automatic footprint than either prior rejection, not a smaller one. Star count
doesn't resolve the structural concern that sank the other two. Rejected for consistency; flagged as
a real candidate worth revisiting once someone actually reads the worker script's data-handling
logic end to end (the same bar `impeccable`'s hook was held to before its install).

Also individually evaluated and rejected: `semgrep` (requires Semgrep account login — fails the
no-new-credentials bar despite otherwise fitting the security workflow), `codspeed` (SaaS,
no attached workflow), `deepeval` (AI-eval framework — plausible future fit for Compass's
`narrate-alerts` feature, but no project runs an eval harness yet), `sourcegraph` (every tracked
project is single-repo, so its core value — cross-repo search — doesn't apply here). Nine
previously-unlisted `wshobson/agents` plugins got individual review and were rejected for overlap
with already-installed capability or no mapped workflow. The remaining unlisted tail in both large
marketplaces (~65 in `wshobson/agents`, ~262 in `claude-plugins-official`) was scanned at a glance
and is overwhelmingly either broad grab-bags duplicating installed skills, domain-specific plugins
with no mapped project, or credentialed third-party SaaS integrations — bulk-logged in
`skills-seen.md` with reasoning rather than itemized row-by-row (see that file for the full
breakdown and the explicit call not to re-scan all ~327 of them line-by-line every week going
forward).

## Token-efficiency / context-management check

Two candidates found; one promoted to a decision, one logged as a not-yet-applicable candidate:

- **Effort-level routing for `Workflow`-dispatched agents** (promoted — see
  `decisions/2026-08-08-token-efficiency-effort-routing.md`). The `Workflow` tool's `agent()` calls
  expose an `effort` parameter (`low`/`medium`/`high`/`xhigh`/`max`) independent of `model`, silently
  defaulted to inherit-from-parent today. Current 2026 guidance (confirmed via web search) recommends
  `xhigh` as the default for coding/agentic work, dropping to `low` only for simple, scoped,
  low-judgment stages — the same shape as the existing Haiku model-routing bullet, applied to a
  second, independent knob. Recommended with the same required boundary: never lower effort for a
  judgment/verification/synthesis stage.
- **Tool Search / `defer_loading`** (Anthropic's Advanced Tool Use feature — lazy-loads MCP/tool
  schemas on demand instead of upfront, ~85% reduction in tool-definition tokens on large catalogs,
  no functional/quality cost since every tool stays fully available, just discovered on demand).
  Confirmed this is *already* active automatically within Claude Code sessions themselves — this
  very session's own tool list shows deferred tools (`CronCreate`, `DesignSync`, the GitHub MCP
  tools) loaded via exactly this mechanism, so there's nothing to change about day-to-day Claude Code
  usage. It only becomes a genuinely new lever if this setup starts *building* an MCP server or an
  Agent-SDK-based tool with a large catalog directly (where `defer_loading: true` would need to be
  set explicitly on tool definitions) — not the case for any project tracked here yet. Logged as a
  candidate below rather than a decision; revisit when that becomes concrete.

## Deep-dive: prompting techniques for agentic coding

Honest finding this week: most of what's currently cited as SOTA prompting technique for agentic
coding turned out to already be reflected in this setup, one way or another — this was mostly a
confirming pass, not a gap-finding one.

- **Spec-driven development** (write the spec/plan first, treat it as the source of truth, let the
  agent implement against it — 2026's dominant framing, explicitly positioned against unstructured
  "vibe coding") — already the working pattern here. This very repo's `specs/` directory is a spec
  written and approved before the corresponding system was built; superpowers'
  `writing-plans`/`executing-plans` skills implement the same discipline for ordinary feature work.
  No gap.
- **Explicit PLAN → ACT → VERIFY → ITERATE loop framing**, **"tell it where to look, what not to
  touch, how to prove success, when to stop" task-scoping** — both already substantially present:
  superpowers' `test-driven-development`/`systematic-debugging`/`verification-before-completion`
  cover the loop; this session's own system prompt already carries explicit scope-matching and
  "don't take actions beyond what was requested" guidance. No gap.
- **Meta-prompting / step-back decomposition (task-understanding → planner → executor roles)** —
  this is what the `Workflow` tool's `phase()`/`pipeline()` primitives and the `Agent` tool's
  subagent-dispatch model already implement structurally; covered in depth by the prior
  agent-architectures deep-dive (`research/2026-08-06-agent-architectures-multi-agent-patterns.md`).
  No new finding here.
- **Effort-level tuning as a prompting/dispatch lever** — genuinely new and actionable; written up
  under token-efficiency above rather than duplicated here, since it's really a delegation-pattern
  concern more than a wording-of-the-prompt one.
- **Structured XML/JSON prompt formatting, few-shot examples, "give explicit permission to express
  uncertainty rather than guess," negative/constraint framing** — general Claude prompting best
  practices (unchanged from Anthropic's standing guidance), already reflected in how this session's
  own tool/skill descriptions are written (e.g. the `Agent` tool's own instructions explicitly model
  "brief the agent like a smart colleague," itself a form of few-shot/explicit-context framing).
  Nothing actionable found specific to *this* setup's config — these are per-task authoring habits,
  not something a standing instruction changes.

No candidate cleared the bar for a `decisions/` entry from this deep-dive specifically — the one
concrete finding (effort routing) is filed under token-efficiency instead, where it actually belongs
as a delegation-pattern lever.

## Candidate improvements (not yet acted on)

| Candidate | Impact | Effort | Notes |
|---|---|---|---|
| Install `context7` | M | L | Recommended this run — `decisions/2026-08-08-install-context7.md` |
| Add effort-level routing to `~/.claude/CLAUDE.md` token-efficiency section | M | L | Recommended this run — `decisions/2026-08-08-token-efficiency-effort-routing.md` |
| Revisit `claude-mem` after reading its worker-service source end to end | M–H (closes the standing cross-project-memory gap) | M (real code-reading effort, not a rubber stamp) | Strongest memory-gap candidate found across all three evaluation rounds (`pensyve`, `remember`, now this) — reputation alone doesn't clear the automatic-hooks concern; needs an actual read |
| Revisit `deepeval` if/when a project ships an evaluatable AI feature (e.g. Compass's `narrate-alerts` gets a regression-testing need) | L now / M–H if it becomes concrete | L | No current attach point; noted so it isn't re-discovered from scratch later |
| Adopt `defer_loading` when this setup ever builds a custom MCP server or Agent-SDK tool with a large catalog | L now (no such project exists) / M if one is built | L | Zero-quality-cost technique, just not applicable to anything currently in scope |
| Deeper look at whether `wshobson/agents`'s bulk-rejected domain-specific plugins are worth a lighter-weight periodic spot-check (vs. never revisiting) | L | L | Purely a process question — is "bulk-reject and move on" the right long-term posture for a 95-plugin marketplace, or should some rotation sample a few each week? Not resolved this run |
