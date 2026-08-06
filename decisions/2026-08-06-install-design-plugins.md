# Decision: install three design-quality plugins

**Date:** 2026-08-06

## Context

User asked for a dedicated research pass on state-of-the-art UI/UX and product design for
AI-assisted development, with instructions to actually integrate high-impact findings, not just
report them (same "research → implement → document" bar as every other lane in this system). See
`research/2026-08-06-design-ux.md` for the full findings this pulled from.

## Decision

Installed three plugins in this persistent environment (not just recommended for later review —
this was live manual work, same standing as the 2026-08-06 three-plugin install):

1. **`frontend-design@claude-plugins-official`** — official Anthropic plugin. Directly addresses
   the single highest-impact finding: AI-generated UI converges on a handful of "generic AI
   aesthetics" defaults unless explicitly constrained. Zero cost, no credentials, no conflict with
   anything installed.
2. **`modern-web-guidance@claude-plugins-official`** — Google Chrome team plugin. 130+ web-platform
   best-practice checks (CSS/layout, forms, performance, accessibility) with no API key (local
   TensorFlow.js semantic search only). Verified reputability before installing: 199 commits, 1.7k
   stars, actively maintained (eval results dated within the last month at check time).
3. **`impeccable@impeccable`** (new marketplace, added same session) — third-party but
   well-vetted: 56k stars, 3.4k forks, known maintainer (Paul Bakaus), explicitly built as a
   companion to `frontend-design` rather than a competitor. Ships `PostToolUse`/`Stop` hooks that
   run on every `Edit`/`Write` — read the hook script (`hook.mjs`, 78 lines) before installing to
   confirm it does purely local, deterministic static analysis (no network calls, no
   `exec`/`spawn`, no LLM/API key) rather than the opaque-automatic-hook pattern that got
   `pensyve`/`remember` rejected earlier the same day (see `skills-seen.md`).

## Alternatives considered

- **Recommend for later review instead of installing now** — rejected: this session *is* a manual
  review pass in a persistent environment (identical position to the earlier `playwright`/
  `block-no-verify`/`claude-code-setup` installs), and the whole point of this pass was "integrate
  what you can," not just log candidates for a future session to act on.
- **Also add a Figma/design-handoff MCP server** — rejected: no current Figma/Canva usage anywhere
  in this setup, Compass's UI is built directly in code rather than from design files. Would be
  unused capability, the exact kind of unnecessary complexity this system is supposed to avoid.
  Logged as explicitly-declined in `research/2026-08-06-design-ux.md` rather than silently omitted,
  so it isn't re-litigated from scratch next time design tooling comes up.

## Reasoning

All three clear the same bar already established for other installs this session: reputable
source, no new credentials, no conflict with what's installed, and a clear mapping to a workflow
actually in use (frontend/UI development). `impeccable`'s hooks got the extra scrutiny an
auto-running-hook plugin deserves — read before install, not assumed safe from stars alone.
