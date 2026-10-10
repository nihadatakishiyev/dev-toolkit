# Decision: recommend a Haiku-routing boundary for the new long-context pricing cliff

**Date:** 2026-10-10 (drafted by the weekly research routine — recommendation, not yet applied to
`~/.claude/CLAUDE.md`)

## Context

`~/.claude/CLAUDE.md`'s token-efficiency section currently has a standing bullet: explicit Haiku
routing for cheap/high-volume subagent work (sharpened in
`decisions/2026-08-06-sharpen-haiku-routing-boundary.md` to exclude anything requiring real
judgment). The bullet's whole premise is that routing to Haiku is a straightforward cost win for
the tasks it targets.

This week's token-efficiency check found that Claude Haiku 5.5 (shipped with Claude Code
v2.1.293, 2026-10-07, now the default Haiku model on the Anthropic API) changes that premise in one
specific way: it carries a **5x per-token price multiplier once a request's prompt passes 100K
tokens**, despite otherwise being priced as a cheap model. Haiku 5.5 is natively 1M-context (no
`[1m]` suffix needed) and is the first Haiku model with its own effort control (starting at
`medium` in Claude Code) — both genuinely good fits for the kind of high-volume, low-judgment
dispatch the existing bullet already targets. But a subagent call that happens to carry a large
prompt (e.g. several full files read into context, or a `fork`-mode dispatch inheriting a long
parent conversation) silently loses the "cheap" assumption past that 100K-token threshold — the
exact inverse of what routing to Haiku is supposed to buy.

Checked whether this is already covered: it isn't. The existing bullet and its sharpened boundary
are both about *when a task is simple/judgment-free enough* to route to Haiku at all; this is an
orthogonal axis — *how large the dispatched prompt is* — that can silently erase the cost benefit
even for a task that correctly clears the existing judgment bar.

## Decision

Recommend adding one clause to the existing Haiku-routing bullet (not a new bullet — this is a
refinement of the same rule, not an independent lever): when routing to Haiku for cost reasons,
keep the dispatched prompt itself small — targeted excerpts/results, not several full files or a
long inherited conversation. If a Haiku-routed task's own prompt would exceed roughly 100K tokens
(e.g., a `fork`-mode dispatch carrying a long parent conversation, or a task that genuinely needs
many/large file reads), either trim what's handed to it or route to a different model instead —
past that size, Haiku 5.5's discount no longer holds and the cost rationale for choosing it
disappears.

**Required boundary:** this is a cost-routing refinement only, not a quality one — it never
overrides the existing "never for judgment work" boundary, and it never means truncating or
skipping content a task actually needs just to stay under the threshold (that would trade a token
saving for a correctness risk, exactly what the quality-first guardrail already forbids). When a
task genuinely needs a large prompt to do its job correctly, the right move is choosing a
different model, not shrinking the prompt below what the task requires.

Not applied to `~/.claude/CLAUDE.md` directly by this run — goes through the same manual-review
gate as every other standing-instruction change.

## Alternatives considered

- **Leave the existing bullet as-is** — simplest, but the bullet's own stated purpose (cost
  savings) silently stops being true past 100K tokens for exactly the kind of context-heavy
  dispatch (`fork` mode, multi-file reads) this account's own practice already uses — same shape of
  gap as every other "lever exists, instruction hasn't caught up" finding this routine has
  previously flagged.
- **Add a wholesale "never route to Haiku for large-context tasks" rule** — rejected as too blunt;
  the existing practice already scopes Haiku routing to small, high-volume, low-judgment work,
  where prompts are typically small by construction. The gap is specifically the intersection with
  `fork`-mode dispatch (which can carry a large inherited conversation into an otherwise-simple
  task) and multi-file-read tasks, not Haiku routing generally — a clause on the existing bullet
  targets that intersection without overclaiming a broader problem.

## Reasoning

Same pattern as every other token-efficiency finding filed here: a real, currently-silent gap
between an existing standing instruction's stated rationale (cost) and a since-changed fact on the
ground (the model it points to now has a pricing cliff the instruction doesn't account for), closed
with an explicit, narrowly-scoped boundary rather than a blanket rule. Zero quality cost — this
only changes *which model/how much prompt* a cost-motivated dispatch uses, never whether a task
gets done or how carefully.
