# Decision: recommend installing the `chrome-devtools-mcp` plugin

**Date:** 2026-08-15 (drafted by the weekly research routine — recommendation, not yet installed)

## Context

This week's skills scan swept `claude-plugins-official`'s current 286-plugin listing for names not
already itemized in `skills-seen.md` (the marketplace grew from 284 to 286 since the last full scan
on 2026-08-08; several previously-unitemized names also surfaced from the earlier bulk-rejected
tail). `chrome-devtools-mcp` — maintained by the Chrome DevTools team itself
(github.com/ChromeDevTools/chrome-devtools-mcp) — lets a coding agent control and inspect a *live*
Chrome browser: record performance traces, read network requests, read console messages with
source-mapped stack traces, and drive the page via Puppeteer.

## Decision

Recommend installing `chrome-devtools-mcp` from `claude-plugins-official` during the next manual
review pass, in a persistent environment where the install actually sticks (this routine's own
cloud sandbox is ephemeral — see `decisions/2026-08-06-fix-skills-autonomy-ephemeral-sandbox.md`).
Not installed by this run itself.

## Alternatives considered

- **Rely on `playwright` alone (already installed)** — considered, but it's a different job:
  Playwright drives browser automation/E2E *tests* against a page; `chrome-devtools-mcp` inspects a
  *live* running browser for debugging (network waterfall, console errors with source maps,
  performance traces) — the thing you actually reach for when a web app misbehaves and you need to
  see what the browser is doing, not run a scripted test suite. Distinct capability, not a
  duplicate; both can stay installed without redundancy the way `playwright` vs. rejected
  `browser-use` was a real either/or.
- **Do nothing, keep debugging by asking the user to paste console/network output** — works today,
  but is strictly worse than the agent reading it directly once a web project needs it.

## Reasoning

Passes all four carve-out criteria: reputable source (official Chrome DevTools team, not a
third-party wrapper), no new credentials (controls a local/already-authenticated Chrome instance),
no conflict with anything installed (distinct from `playwright`'s test-automation role, see above),
and maps to a workflow already in use — every web project here (Compass) benefits from live
browser-debugging access during UI work, and this is the first tool identified that provides that
specific capability.
