# Decision: recommend installing the `playwright` plugin

**Date:** 2026-08-06 (drafted by the weekly research routine — recommendation, not yet installed)

## Context

`baseline.md`'s "Browser E2E testing" section notes a real gap: no `chromium-cli` in sandboxed/cloud
sessions, and Compass's Playwright-based REPL (`scripts/browser/`) was committed to that project
specifically rather than generalized into a reusable cross-project tool. This week's skills scan
(official `claude-plugins-official` marketplace) surfaced `playwright` — a browser automation/E2E
testing MCP server authored by Microsoft, bundled as one of the official marketplace's partner
plugins.

## Decision

Recommend installing `playwright` from `claude-plugins-official` during the next manual review pass,
in a persistent environment where the install actually sticks (this routine's own cloud sandbox is
ephemeral — see `decisions/2026-08-06-fix-skills-autonomy-ephemeral-sandbox.md`). Not installed by
this run itself.

## Alternatives considered

- **`browser-use`** (also in `claude-plugins-official`) — same category (give Claude a real browser),
  but no reason to run two competing browser-automation plugins for the same gap. Playwright is the
  more established, first-party-partner option (Microsoft-authored, widely adopted testing
  framework) — rejected `browser-use` in `skills-seen.md` on that basis.
- **Keep generalizing Compass's own REPL script instead** — would work, but re-solves a problem an
  official, maintained MCP server already solves, and ties the tooling to one project's bespoke
  script rather than a reusable cross-project skill.

## Reasoning

Passes all four carve-out criteria cleanly: reputable source (Microsoft, official marketplace),
no new credentials/secrets, no conflict with anything already installed (no browser/E2E skill
exists yet), and it directly maps to a workflow already in active use (Compass's browser E2E
testing needs). This is the clearest candidate found this run precisely because it closes a gap
`baseline.md` already named explicitly rather than introducing a new one speculatively.
