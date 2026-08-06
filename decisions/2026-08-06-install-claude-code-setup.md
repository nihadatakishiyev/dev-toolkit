# Decision: recommend installing the `claude-code-setup` plugin

**Date:** 2026-08-06 (drafted by the weekly research routine — recommendation, not yet installed)

## Context

This week's skills scan of `claude-plugins-official` surfaced `claude-code-setup`, a first-party
Anthropic plugin that analyzes a codebase and recommends tailored Claude Code automations — hooks,
skills, MCP servers, subagents — for it. This overlaps conceptually with what this very repo's
design already calls for during the "manual implementation pass ... starting a new project", which
today is done by hand, one project at a time.

## Decision

Recommend installing `claude-code-setup` from `claude-plugins-official` during the next manual
review pass, in a persistent environment. Not installed by this run itself.

## Alternatives considered

- **Keep doing this analysis by hand per-project** — status quo; works, but is exactly the kind of
  repeatable analysis a tool should shoulder so the manual pass can spend its attention on judgment
  calls (what to actually install/change) rather than on rediscovering what's available every time.
- **Treat it as a replacement for `dev-toolkit`'s own manual review pass** — rejected as framing:
  this plugin analyzes one codebase in isolation; it has no notion of this repo's accumulated
  decisions/research or cross-project consistency. It's a complement that speeds up the
  per-project analysis step, not a substitute for the durable, cross-project record this repo
  keeps.

## Reasoning

Passes the four carve-out criteria: first-party Anthropic source, no new credentials, no conflict
with anything installed, and it maps to the "starting a new project" workflow this system's own
design already treats as a first-class trigger for the manual review pass.
