# Decision: install the 3 plugins the routine recommended

**Date:** 2026-08-06

## Context

The weekly routine's second run recommended 3 plugins (`playwright`, `block-no-verify`,
`claude-code-setup`) rather than installing them, per
`decisions/2026-08-06-fix-skills-autonomy-ephemeral-sandbox.md` — its cloud sandbox is ephemeral,
so the actual install has to happen during a manual review pass in a persistent environment. This
is that pass, done the same day since the recommendations were fresh and the reasoning behind each
was already fully drafted (see `decisions/2026-08-06-install-playwright.md`,
`decisions/2026-08-06-install-block-no-verify.md`, `decisions/2026-08-06-install-claude-code-setup.md`).

## Decision

Independently re-verified each plugin actually exists with the described name/version/description
(not just trusting the drafted recommendation at face value) by adding the `wshobson/agents`
marketplace here (as `claude-code-workflows`) and reading both marketplaces' `marketplace.json`
directly. All three matched exactly. Installed all three via `claude plugin install
<name>@<marketplace>`, user scope, this environment.

## Alternatives considered

None beyond what's already in the three individual recommendation docs — this pass is about
executing an already-reasoned decision, not re-litigating it. Re-verification (rather than blind
trust) was the only addition at this step.

## Reasoning

This is the manual-review half of the system doing exactly its designed job: turning a
routine-drafted recommendation into an actual, durable change, on a human-supervised timescale
rather than an autonomous one. Doing it same-day here rather than deferring is just convenience —
nothing about the design requires it to happen immediately.
