# Decision: sharpen the Haiku-routing boundary + separate it from product-code model choices

**Date:** 2026-08-06

## Context

While reviewing my own Phase 0 implementation plan for the Compass redesign (before execution,
per the user's request to critically self-review it), I'd applied the existing Haiku-routing
standing instruction to justify a *product* engineering decision — which model a new Edge Function
(`narrate-alerts`) should call at runtime to narrate a financial insight — by analogy to the
existing subagent-dispatch rule. The user pushed back: the instruction should be framed as applying
only to genuinely repetitive, simple tasks, explicitly not to complete redesigns or critical
implementations.

## Decision

Two changes to `~/.claude/CLAUDE.md`'s Haiku-routing bullet:

1. Added explicit negative examples alongside the existing positive ones: never route a complete
   redesign, a critical implementation, or anything requiring real judgment to a lighter model —
   even if one piece of that larger work looks simple in isolation.
2. Added a new bullet clarifying that the rule governs *my own* subagent dispatch specifically —
   it does not automatically extend to a *product's own* AI-feature model choice (e.g. an Edge
   Function calling the Anthropic API). That's a separate engineering decision with its own stakes
   and needs its own justification each time, argued on its own merits, not borrowed by analogy
   from this bullet.

The `narrate-alerts` Haiku choice itself was left in place in the Phase 0 plan (not reverted) — on
reflection it still holds up on its own merits under the new, tighter framing: the actual financial
computation stays entirely in deterministic Postgres functions, untouched by any model; Haiku only
rewrites an already-computed, already-verified structured payload into one constrained sentence
("never invent numbers"), a genuinely repetitive/bounded/low-stakes task, not a critical
implementation. This is now argued explicitly on its own terms in the plan rather than by appeal to
the subagent-dispatch rule.

## Alternatives considered

- **Revert narrate-alerts to a stronger model** — rejected: the user's feedback was about how the
  *rule* should be framed, not a specific instruction to undo that one choice, and the choice
  survives scrutiny under the tightened framing since the critical (numeric) part of the work was
  never delegated in the first place.
- **Leave the rule as-is, just be more careful applying it** — rejected: the whole point of a
  written standing instruction is that it should hold up without requiring the same judgment call
  to be re-litigated every time; an instruction that only works when applied carefully is exactly
  the kind of gap [[quality-first-guardrail]] already exists to close for this same section.

## Reasoning

The underlying mistake was scope creep — reaching for an existing, adjacent-sounding rule to justify
a decision it wasn't actually written to cover, rather than arguing that decision on its own terms.
The fix is the same shape as [[quality-first-guardrail]]: an efficiency-flavored instruction needs
its boundaries stated explicitly, both what it covers and what it doesn't, or it quietly gets
stretched further than intended by whoever's using it next — including me.
