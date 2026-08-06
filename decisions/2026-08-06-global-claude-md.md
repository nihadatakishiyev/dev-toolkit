# Decision: establish `~/.claude/CLAUDE.md` as the standing-instruction surface

**Date:** 2026-08-06

## Context

User asked for continuous improvement across the whole operating environment/workflow, not just
skills, plus a dedicated focus on token efficiency. Researching that surfaced a structural gap:
`dev-toolkit` durably records *why* a decision was made, but nothing automatically re-reads it at
the start of a session the way a project's own `CLAUDE.md` does. A behavioral decision (e.g. "route
simple subagent work to a cheaper model") had no mechanism to actually change future behavior —
only plugin installs did, since those persist through `~/.claude/plugins` itself. `baseline.md` had
already flagged "no global `CLAUDE.md`" as a gap since this repo's creation; this is what finally
gave that gap a concrete reason to get fixed.

## Decision

Created `~/.claude/CLAUDE.md` (global — Claude Code loads a user-level `CLAUDE.md` in every
project's session, not just a project-local one). `dev-toolkit` stays the reasoning/history record;
this file is the short, currently-active summary distilled from it. First two entries: explicit
model routing for cheap/high-volume subagent work (Haiku instead of inheriting the parent model),
and a reminder to actually use the existing per-project memory system rather than let it sit empty
(the same discipline gap noted in `baseline.md` since day one of this repo).

## Alternatives considered

- **Rely on `dev-toolkit` alone, re-read it manually each session** — rejected: exactly the failure
  mode this fixes. Nothing forces a re-read, so a filed decision has no actual effect until someone
  happens to look.
- **Put standing instructions in each project's own `CLAUDE.md`** — rejected as the sole mechanism:
  works for project-specific conventions (and should still be used for those), but duplicates
  cross-project instructions into every repo and drifts out of sync between them. Global belongs at
  the global level.

## Reasoning

This is infrastructure for every future decision this system makes, not just the two entries in it
today — any future `dev-toolkit` decision that's meant to change how Claude actually operates
(rather than what's installed) now has somewhere real to land.
