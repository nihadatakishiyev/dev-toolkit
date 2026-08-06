# Decision: skills carve-out logs a recommendation, doesn't install in-session

**Date:** 2026-08-06 (supersedes part of `decisions/2026-08-06-skills-autonomy.md`)

## Context

The first real run of the weekly routine (see `research/2026-08-06-mcp-servers-integrations.md`)
found that the scheduled routine's cloud session is genuinely ephemeral — a fresh sandbox with no
persisted `~/.claude/plugins`. `claude plugin install` run inside that session only affects that
session; it's gone when the sandbox tears down. The original auto-install carve-out assumed
"installed" meant durably installed in the user's actual environment — it doesn't, for this
specific execution context.

## Decision

The skills lane still auto-*decides* (evaluates against the same four criteria, no review
checkpoint needed for that judgment) — but for a **clearly beneficial** skill, the routine now logs
it as a strong recommendation (`skills-seen.md` verdict `recommended`, a `decisions/` entry drafted
in advance, a `CHANGELOG.md` line) rather than running `claude plugin install` itself. The actual
install command runs during the next manual review pass, in a persistent environment where it
sticks. A **rejected** skill is unaffected by this change — rejection was always just a log entry,
never an install, so there was nothing ephemeral about it to begin with.

## Alternatives considered

- **Leave it as originally speced** (routine installs it directly) — rejected: verified not to
  work as intended; would keep "succeeding" at installing something that vanishes every time,
  silently providing zero actual value while looking like it's working.
- **Persist `~/.claude/plugins` across routine runs somehow** (e.g. via a cache/volume) — not
  pursued: adds real infrastructure complexity for a problem that "log a recommendation" solves
  more simply, and installing plugins is cheap/fast enough that redoing it once during manual
  review isn't a meaningful cost.

## Reasoning

This is a case where designing on paper got the mechanism wrong in a way that only showed up by
actually running it once — which is exactly why the design's testing/verification section insisted
the first real run *is* the verification step, not a formality. The fix keeps the original intent
(skills don't need a human judgment call to be evaluated) while correcting the part that was wrong
(where the resulting install command actually needs to execute).
