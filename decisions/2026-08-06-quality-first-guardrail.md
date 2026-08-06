# Decision: token-efficiency techniques require an explicit quality boundary

**Date:** 2026-08-06

## Context

Right after `~/.claude/CLAUDE.md` was created with its first token-efficiency entries (Haiku
routing for subagents, targeted reads), user feedback: don't sacrifice quality for token
efficiency — low-quality work costs more later (rework, bugs, bad decisions) than tokens saved now.

## Decision

Added an explicit "quality first" framing at the top of `~/.claude/CLAUDE.md`'s "Token efficiency"
section, and reworded both existing entries to state their own boundary (Haiku routing: only for
work that's *genuinely* simple, default to inheriting the parent model when task difficulty is
unclear rather than guessing; targeted reads: read more the moment there's real uncertainty, never
guess to save tokens). Also added this as a standing requirement in the design spec itself
(`specs/2026-08-06-continuous-self-improvement-design.md`) — every future entry the weekly routine
proposes for this section must state when the technique should *not* apply, not just when it
should.

## Alternatives considered

- **Just remove the token-efficiency section** — rejected: the underlying techniques (model
  routing, targeted reads) are real and correct when applied to genuinely simple work; the problem
  was the instruction had no stated boundary, not that it was wrong to have one.
- **Leave it as general good judgment, no explicit boundary written down** — rejected: this is
  exactly the gap that caused the concern in the first place. An instruction that only says "do X
  to save tokens" reads as unconditional over time, especially once other instructions get added
  around it by future automated passes that don't have this conversation's context.

## Reasoning

A cost-saving instruction with no stated limit is a liability precisely because it's designed to be
followed by default — the fix isn't caution about writing efficiency instructions at all, it's
making sure each one is honest about its own edges.
