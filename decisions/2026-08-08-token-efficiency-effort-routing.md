# Decision: recommend adding explicit effort-level routing to the token-efficiency section

**Date:** 2026-08-08 (drafted by the weekly research routine — recommendation, not yet applied to
`~/.claude/CLAUDE.md`)

## Context

`~/.claude/CLAUDE.md`'s token-efficiency section currently covers *which model* a subagent runs on
(explicit Haiku routing for cheap/high-volume work, sharpened in
`decisions/2026-08-06-sharpen-haiku-routing-boundary.md` to exclude anything requiring real
judgment). It says nothing about *reasoning effort*, a separate, independent knob. The `Workflow`
tool's `agent()` calls expose an `effort` option (`low`/`medium`/`high`/`xhigh`/`max`) that defaults
to inheriting the session's effort level when omitted — mirroring how `model` defaults to
inheriting the parent model. Web research this week confirms this is an active, documented
technique industry-wide as of 2026: Anthropic's own effort-level guidance recommends `xhigh` as the
default for coding/agentic use cases, `high` as the floor for intelligence-sensitive work, `medium`
for cost-sensitive lower-complexity tasks, and `low` only for simple, scoped tasks where latency
matters more than reasoning depth — explicit permission to spend *less* compute on genuinely easy
stages, not a general license to cut corners.

Checked whether this is already platform-handled and doesn't need a standing instruction: it isn't.
Unlike prompt caching and auto-compaction (both automatic, no user lever), effort level is a
per-call choice that's silently defaulted to "inherit" today — every `Workflow` agent dispatch this
session and prior sessions has run at the parent's effort level even for mechanical, low-judgment
stages, the same gap the Haiku-routing decision found for `model` two weeks ago.

## Decision

Recommend adding a new bullet to `~/.claude/CLAUDE.md`'s token-efficiency section: when dispatching
agents via the `Workflow` tool, set `effort` explicitly per stage rather than leaving it at the
inherited default — `low` for mechanical/high-volume stages with no real judgment call (formatting
checks, simple lookups, single-lens classification), `high`/`xhigh` for stages doing genuine
reasoning, verification, or judgment (adversarial-verify passes, judge panels, synthesis). This is
the same discipline as the existing Haiku-routing bullet, applied to a different, independent axis
— model and effort are separate knobs and both should be set deliberately, not just one of them.

**Required boundary (per the quality-first guardrail already governing this section):** never lower
effort for a stage that involves real judgment, correctness verification, or synthesis across
multiple findings — the same carve-out the Haiku-routing bullet already states for model choice.
When in doubt, leave effort at its inherited default rather than guessing low. This only applies to
`Workflow`-based `agent()` dispatch, which exposes the parameter — the plain `Agent` tool has no
`effort` parameter to set, so there's nothing to change about how that tool is used.

Not applied to `~/.claude/CLAUDE.md` directly by this run — goes through the same manual-review
gate as every other standing-instruction change.

## Alternatives considered

- **Leave effort at the inherited default always** — simplest, but leaves a real, low-risk cost
  lever unused for exactly the kind of high-volume/low-judgment stages the Haiku-routing bullet
  already identifies as a target (they're often the same stages) — same shape of gap, just on a
  different parameter.
- **Fold this into the existing Haiku-routing bullet rather than a new one** — considered, but
  model and effort are genuinely independent choices (a stage can reasonably be low-effort on a
  frontier model, or full-effort on Haiku for a narrow, well-specified check); keeping them as
  separate, individually-scoped bullets avoids implying they always move together.

## Reasoning

Same pattern as `decisions/2026-08-06-token-efficiency-and-agent-landscape.md`'s original
Haiku-routing finding: a real, currently-unused lever exposed directly by this harness's own tools,
scoped with an explicit not-applicable boundary so it can't drift into "always cut reasoning" the
way the quality-first guardrail was written to prevent. Zero quality cost when the boundary holds —
this only ever reduces spend on stages that were never going to benefit from more reasoning in the
first place.
