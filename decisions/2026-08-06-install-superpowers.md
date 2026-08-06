# Decision: install the superpowers plugin

**Date:** 2026-08-06

## Context

User explicitly requested "superpowers" during Compass work, identified as `obra/superpowers`
(github.com/obra/superpowers) once confirmed by name.

## Decision

Added the `superpowers-dev` marketplace and installed the `superpowers` plugin (v6.2.0, user
scope). 14 skills: brainstorming, dispatching-parallel-agents, executing-plans,
finishing-a-development-branch, receiving-code-review, requesting-code-review,
subagent-driven-development, systematic-debugging, test-driven-development, using-git-worktrees,
using-superpowers, verification-before-completion, writing-plans, writing-skills.

## Alternatives considered

None — user named the specific plugin directly, so this wasn't a comparative evaluation. Recorded
here mainly so `skills-seen.md` has a decision to point to, and so future research passes don't
re-surface `superpowers-dev` as a "newly discovered" marketplace.

## Reasoning

Direct user request. The skill set (TDD, systematic debugging, planning, code review workflows)
maps cleanly onto how we already work, which is itself the reason this whole toolkit repo exists —
using it to plan and build itself is the first real dogfooding of the system.
