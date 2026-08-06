# Decision: create dev-toolkit as a passive changelog/decision-log repo

**Date:** 2026-08-06

## Context

While working on Compass, installing the `superpowers` plugin mid-session prompted a broader
request: Claude's own setup (skills, MCP servers, config, workflows) should improve deliberately
over time, across every project, not be rediscovered fresh each time. That needs somewhere durable
to live — not tied to any one client project's repo.

## Decision

A dedicated, standalone git repo (`dev-toolkit`), passive (documents changes, doesn't manage the
live config itself via symlinks/sync), with a weekly scheduled research routine that logs findings
and only auto-acts on the low-risk skills lane (see `decisions/2026-08-06-skills-autonomy.md`).
Full reasoning and architecture: `specs/2026-08-06-continuous-self-improvement-design.md`.

## Alternatives considered

- **Inside `~/.claude/` directly** — rejected: that directory isn't normally version-controlled,
  weaker history/rollback.
- **Active dotfiles repo** (symlinked into `~/.claude/`) — rejected: more moving parts, real risk
  of ever breaking live config for marginal benefit over a passive log at this stage. Could
  revisit later if the passive approach turns out to be insufficient.
- **Inside the Compass repo** — rejected: conflates a personal finance app's repo with cross-project
  meta-tooling; wouldn't apply to future non-Compass projects.

## Reasoning

Git history on the decision/research files themselves already gives most of the value of a full
dotfiles system (diffable, revertible, timestamped) without the risk of accidentally breaking
`~/.claude/` through sync tooling. Can graduate to the active dotfiles model later if the split
between "documented reasoning" and "actual config" ever causes real drift or confusion in
practice — not needed preemptively.
