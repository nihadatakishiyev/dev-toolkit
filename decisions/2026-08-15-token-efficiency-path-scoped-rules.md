# Decision: recommend documenting `.claude/rules/` path-scoped instructions as a token-efficiency technique

**Date:** 2026-08-15 (drafted by the weekly research routine — recommendation, not yet applied to
`~/.claude/CLAUDE.md`)

## Context

This week's deep-dive (memory systems — see `research/2026-08-15-memory-systems.md`) read Claude
Code's official memory docs (`code.claude.com/docs/en/memory`) end to end while comparing against
`baseline.md`'s existing description of the memory system. Alongside confirming the already-known
auto-memory mechanism, it surfaced a mechanism not previously documented anywhere in this repo:
`.claude/rules/` — markdown files scoped to specific files via a `paths:` frontmatter glob (e.g.
`paths: ["src/api/**/*.ts"]`), loaded into context only when Claude actually reads a matching file,
as opposed to `CLAUDE.md` content, which loads in full at every session start regardless of whether
it's relevant to that session's work. A user-level variant (`~/.claude/rules/`) applies the same
mechanism globally, the same scope `~/.claude/CLAUDE.md` operates at.

Checked whether this is already covered by the existing token-efficiency bullets (Haiku model
routing, targeted reads over whole-file reads, the drafted-but-unapplied effort-routing bullet from
`decisions/2026-08-08-token-efficiency-effort-routing.md`): it isn't — those all govern per-call
choices during a session; this is about *where a standing instruction lives*, a structural choice
made once when writing the instruction, not a runtime decision.

## Decision

Recommend adding a new bullet to `~/.claude/CLAUDE.md` (or, more precisely, a new short paragraph
near the token-efficiency section): when a standing instruction only matters for a specific file
type or directory (e.g. "when editing CSS, do X", "API handlers must do Y"), prefer writing it as a
path-scoped rule in `~/.claude/rules/<topic>.md` (`paths:` frontmatter) over adding it to
`~/.claude/CLAUDE.md` directly — it only enters context on the sessions that actually touch matching
files, instead of consuming tokens on every session regardless of relevance.

**Required boundary (per the quality-first guardrail already governing this section):** this only
applies to instructions that are genuinely conditional on file type/location. Do **not** path-scope
anything that must hold regardless of which file is touched, or that needs to be in effect from the
start of a task before any specific file is opened — git-safety rules, security posture, "always
verify before claiming done"-style workflow rules, and anything else in this section's existing
bullets all stay in `~/.claude/CLAUDE.md` itself. Path-scoped rules trigger when Claude *reads a
matching file*, not on every tool use, so anything that needs to be enforced independent of file
access (or before the relevant file is ever opened) would silently fail to load in time if moved
here. When in doubt whether an instruction is genuinely file-scoped, leave it in `CLAUDE.md`.

Not applied to `~/.claude/CLAUDE.md` directly by this run — goes through the same manual-review gate
as every other standing-instruction change. Also not urgent to act on immediately: `baseline.md`
doesn't describe today's `~/.claude/CLAUDE.md` as large enough yet to need this (three short
sections). This is worth having on file as a technique for when it does grow, rather than something
to force onto a currently-small file.

## Alternatives considered

- **Leave everything in `~/.claude/CLAUDE.md` regardless of size** — simplest, and fine at today's
  size, but is exactly the failure mode the official docs warn about directly ("Files over 200 lines
  consume more context and may reduce adherence") — worth having the escape valve documented before
  it's needed under time pressure.
- **Recommend moving something specific today** (e.g. the UI/UX section, which baseline already
  notes is "scoped to real UI/product surfaces") — considered, but that section is already
  reasonably short and the UI/UX scoping is currently just a sentence, not enough content to justify
  the split yet. Revisit if that section grows.

## Reasoning

Same shape as the effort-routing recommendation: a real, currently-unused lever exposed directly by
this harness (not a third-party workaround), scoped with an explicit not-applicable boundary so it
can't drift into hiding load-bearing instructions where they might not reliably fire. Zero quality
cost when the boundary holds — a correctly-scoped path rule fires exactly when it's relevant, same
as if it were always loaded, and consumes zero tokens on sessions where it wouldn't have mattered
anyway.
