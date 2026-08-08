# Decision: recommend installing the `context7` plugin

**Date:** 2026-08-08 (drafted by the weekly research routine — recommendation, not yet installed)

## Context

This week's skills scan re-inspected `claude-plugins-official`'s full 284-plugin listing (up from
278 at the last full scan) and surfaced `context7` — an Upstash-authored MCP server for
version-specific library documentation lookup, connecting to Context7's hosted remote MCP endpoint
(no local Node.js/`npx` required). Per web search, it's specifically called out as "the plugin most
experienced Claude Code users install first," with 348k+ installs recorded in Anthropic's public
plugin directory as of June 2026.

## Decision

Recommend installing `context7` from `claude-plugins-official` during the next manual review pass,
in a persistent environment where the install actually sticks (this routine's own cloud sandbox is
ephemeral — see `decisions/2026-08-06-fix-skills-autonomy-ephemeral-sandbox.md`). Not installed by
this run itself.

## Alternatives considered

- **Rely on ad hoc `WebSearch`/`WebFetch` for library docs instead** — works, but pulls
  general-web-search-quality results rather than version-pinned documentation/code examples sourced
  directly from the library's own repository; noisier and less reliable for "what's the current API
  for library X at version Y" questions that come up constantly in coding sessions.
- **`sourcegraph`** (also newly reviewed this run) — different problem (cross-repo code search
  within your own codebases, not external library documentation); not a substitute, and separately
  rejected below for likely needing a paid multi-repo subscription to be useful here anyway.

## Reasoning

Passes all four carve-out criteria: reputable source (Upstash, official-marketplace-curated,
very high install count), no new credentials/secrets required (marketplace listing confirms it
"works anonymously out of the box; set `CONTEXT7_API_KEY` for higher rate limits" — the key is
optional, not a gate), no conflict with anything already installed (nothing currently does library
documentation retrieval), and it maps to a workflow genuinely in use on every coding session —
looking up how a library/framework actually behaves before writing code against it.
