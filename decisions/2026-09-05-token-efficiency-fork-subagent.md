# Decision: recommend a `fork`-vs-fresh-subagent delegation rule for the token-efficiency section

**Date:** 2026-09-05 (drafted by the weekly research routine — recommendation, not yet applied to
`~/.claude/CLAUDE.md`)

## Context

This week's token-efficiency check found that the `Agent` tool in this harness supports a
`subagent_type: "fork"` mode, distinct from every other named subagent type
(`claude`/`Explore`/`general-purpose`/`Plan`/etc.). A fork inherits the entire parent conversation
instead of starting cold — same system prompt, same tools, same model (a fork always inherits the
parent model; the `model` override parameter is explicitly ignored for it), same history — so its
first request reuses the parent's prompt cache instead of paying to re-establish context from
scratch. Its own tool calls still stay out of the parent's conversation; only its final result
returns, same containment property as a regular subagent.

This is not yet documented anywhere in `baseline.md`'s "Agent architecture" section, which lists
only the platform-provided named agent types. Web research (2026-03 sources) confirms this is a
deliberate, documented harness feature ("Context Fork") aimed at exactly this cost profile: cheaper
than a fresh subagent when the subtask genuinely needs the accumulated context, because a fresh
subagent has to be told everything it needs via its prompt (file paths, prior errors, decisions
made so far) and cannot see the conversation at all.

Checked whether this is already covered by an existing bullet: it isn't. The existing
Haiku-routing and effort-routing bullets are about *which model/how much reasoning* a subagent
uses; this is a third, independent axis — *how much prior context a subagent starts with* — with
its own distinct cost/quality tradeoff.

## Decision

Recommend adding a bullet to `~/.claude/CLAUDE.md`'s token-efficiency section: prefer
`subagent_type: "fork"` over a fresh subagent (`general-purpose`, etc.) when, and only when, the
delegated subtask genuinely needs the accumulated conversation context to do its job well — e.g.
continuing an in-progress investigation, or a side task that depends on decisions/state already
established this session. In that situation, forking avoids both the token cost of re-explaining
context in the prompt and the risk of leaving something out that a fresh subagent would need
restated.

**Required boundary:** do NOT use `fork` (a) for adversarial or independent verification, where
the entire point is that the checker should reach a conclusion without seeing — and being
anchored by — the reasoning that produced the thing being checked; a fresh subagent with a narrow,
self-contained prompt is what makes that check meaningful; or (b) when the parent conversation is
long and mostly irrelevant to the subtask, since a fork still carries that bulk into the subagent's
context and can dilute its focus even though the tokens are cache-cheap — a fresh subagent with a
concise, targeted brief is both cheaper and sharper in that case. When unsure whether the subtask
needs prior context, default to a fresh subagent with an explicit brief rather than forking — an
under-briefed fresh subagent fails loudly (missing context, asks or guesses wrong) where an
unnecessarily forked one just quietly wastes some tokens and focus, but the fresh-subagent failure
mode is the one worth designing against since it's a correctness risk, not just a cost one.

Not applied to `~/.claude/CLAUDE.md` directly by this run — goes through the same manual-review
gate as every other standing-instruction change. `baseline.md`'s "Agent architecture" section is
also stale on this point (doesn't list `fork` as an available subagent type at all) and should be
updated during the next pass that touches that section.

## Alternatives considered

- **Always use fresh subagents, ignore `fork` exists** — simplest, but leaves a real, zero-quality-
  cost lever unused for the specific case (continuing in-progress work, side tasks needing prior
  state) where restating context in a prompt is itself wasted tokens and a source of omission risk.
- **Recommend `fork` as the general default for all delegation** — rejected: the verification-
  independence and context-dilution cases above are real quality risks, not hypothetical ones, so a
  blanket default would violate the "state when it should NOT apply" requirement for anything added
  to this section.

## Reasoning

Same shape as the two prior token-efficiency recommendations (Haiku model routing, effort-level
routing): a real, currently-undocumented lever exposed directly by this harness's own tooling,
recommended only with an explicit not-applicable boundary attached so it can't drift into "fork
everything" over time. Zero quality cost when the boundary holds — the boundary specifically
carves out the two situations (independent verification, irrelevant bulk) where forking would
actually hurt rather than just fail to help.
