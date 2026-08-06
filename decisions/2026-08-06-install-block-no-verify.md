# Decision: recommend installing the `block-no-verify` plugin

**Date:** 2026-08-06 (drafted by the weekly research routine — recommendation, not yet installed)

## Context

This week's skills scan of the `wshobson/agents` marketplace surfaced `block-no-verify` (v1.0.1) —
a small `PreToolUse` hook that blocks `--no-verify`, `--no-gpg-sign`, and other git-hook-skipping
flags from being run by an agent. This is already a rule Claude is instructed to follow (the
harness's own git-safety guidance says never to skip hooks or bypass signing unless explicitly
asked), but that's currently enforced only by the model reading and following the instruction —
there's no hard stop if it doesn't.

## Decision

Recommend installing `block-no-verify` from `claude-code-workflows` (`wshobson/agents`) during the
next manual review pass, in a persistent environment. Not installed by this run itself.

## Alternatives considered

- **Rely on the existing prompt-level instruction alone** — status quo; works in the overwhelming
  majority of cases but has no enforcement layer if a session goes off-script under pressure (e.g.
  a stuck pre-commit hook and an impatient retry).
- **Write a custom hook instead of installing a plugin** — plausible, but `block-no-verify` already
  does exactly this, is small enough to audit in full before installing, and there's no reason to
  hand-roll a one-file hook a maintained plugin already provides.

## Reasoning

Passes the four carve-out criteria: single-purpose and small enough to review in full (auditable),
no new credentials, no conflict with anything installed (baseline's only current hook is
superpowers' `SessionStart`), and it maps directly to a rule already governing every git workflow
in this account's Claude Code usage. Converts a prompt-level instruction into an enforced one at
essentially zero cost.
