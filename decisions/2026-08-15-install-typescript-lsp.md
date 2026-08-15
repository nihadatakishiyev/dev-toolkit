# Decision: recommend installing the `typescript-lsp` plugin

**Date:** 2026-08-15 (drafted by the weekly research routine — recommendation, not yet installed)

## Context

Same sweep as `decisions/2026-08-15-install-chrome-devtools-mcp.md` surfaced `typescript-lsp` — a
first-party plugin bundled directly in `claude-plugins-official`'s own `plugins/` directory (not an
external wrapper), providing a TypeScript/JavaScript language server for code intelligence
(type-aware diagnostics, go-to-definition, and similar LSP-backed navigation) inside the coding
loop. The marketplace listing includes a matching family of per-language LSP plugins
(`pyright-lsp`, `rust-analyzer-lsp`, `gopls-lsp`, `clangd-lsp`, `csharp-lsp`, `kotlin-lsp`,
`ruby-lsp`, `php-lsp`, `swift-lsp`, `jdtls-lsp`, `liquid-lsp`) — evaluated together, `typescript-lsp`
is the only one with a clearly mapped current project: Compass is a Next.js/TypeScript codebase
(per `baseline.md`'s design-tooling notes).

## Decision

Recommend installing `typescript-lsp` from `claude-plugins-official` during the next manual review
pass, in a persistent environment where the install actually sticks. Not installed by this run
itself. The rest of the per-language LSP family is rejected below (logged in `skills-seen.md`) for
having no mapped current-project language — same "no clearly mapped workflow" bar used to reject
several `wshobson/agents` language-specific plugins on 2026-08-08.

## Alternatives considered

- **`serena`** (also newly reviewed this run, `./external_plugins/serena`) — broader multi-language
  semantic-code MCP server built on LSP integration, also plausible for this gap. Rejected in favor
  of `typescript-lsp` specifically: Compass is single-language (TypeScript), so the narrower,
  first-party, single-purpose plugin is the better fit than standing up a heavier multi-language
  server for one language actually in use — same "pick one, not both" logic as the
  `playwright`-vs-`browser-use` call on 2026-08-06. Revisit `serena` if a second project introduces
  a different primary language.
- **No LSP integration at all** — status quo; works, but leaves type-aware navigation/diagnostics to
  ad hoc grep and manual `tsc` runs instead of a purpose-built server.

## Reasoning

Passes all four carve-out criteria: reputable source (first-party, bundled directly in the official
marketplace repo, not a third-party fork), no new credentials (local language server, no API keys),
no conflict with anything installed (no existing skill provides LSP-backed code intelligence), and
maps directly to Compass's actual stack rather than a hypothetical one.
