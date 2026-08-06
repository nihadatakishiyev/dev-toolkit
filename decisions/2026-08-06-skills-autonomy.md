# Decision: skills get auto-install autonomy; everything else stays manual

**Date:** 2026-08-06

## Context

The overall system defaults to "research only, manual implementation" — nothing changes live
config without a review checkpoint. Skill discovery was explicitly called out as wanting to be
continuous, which raised whether it should be an exception to that default.

## Decision

Installing a newly discovered skill is auto-applied during the weekly routine when it meets all
of: reputable source, no new credentials/secrets required, no conflict with an already-installed
skill, and a clear match to a workflow actually in use. Retiring or replacing an existing skill is
**not** auto-applied — it's logged as a candidate for manual review, same as every other kind of
change.

## Alternatives considered

- **Everything about skills queues for review too** (fully consistent with the general rule) —
  rejected: would leave "continuous" skill discovery no more continuous than anything else,
  defeating the specific ask.
- **Fully autonomous: install AND retire/replace on its own** — rejected: retiring a skill already
  in use is more consequential than adding a new, reversible one (something might depend on it in
  ways that aren't obvious from the outside), so it doesn't get the same low-risk treatment.

## Reasoning

Installing a skill is close to the safest possible autonomous action available: it's additive,
fully reversible (`claude plugin uninstall`), touches no credentials, and can't silently break
anything already working. Removing one doesn't share that safety margin, so it gets held to the
same bar as every other kind of infrastructure change.
