# Decision: add a "UI/UX & product design" section to `~/.claude/CLAUDE.md`

**Date:** 2026-08-06

## Context

The design research pass (`research/2026-08-06-design-ux.md`) surfaced two kinds of findings:
things solved by installing a plugin (done, see `decisions/2026-08-06-install-design-plugins.md`),
and things that are workflow *habits* with no tool to install — a self-critique screenshot loop,
an accessibility-by-default checklist, a quick heuristic self-review pass. Per the precedent set by
`decisions/2026-08-06-global-claude-md.md`: a behavioral decision only actually changes future
sessions if it lands in `~/.claude/CLAUDE.md`, not just this repo.

## Decision

Added a "UI/UX & product design" section to `~/.claude/CLAUDE.md` with four defaults, each scoped
to real UI/product surfaces (not throwaway scripts or internal debug tooling — matches the
"quality first" boundary discipline already required of the token-efficiency section, applied here
to a different axis: don't over-invest polish where it doesn't matter, but don't skip it by default
where it does):

1. Invoke the `frontend-design` skill for non-trivial UI work rather than defaulting to
   unstyled/generic Tailwind output.
2. Apply the WCAG 2.2 AA accessibility floor by default (keyboard focus visible, 24×24px minimum
   target size, ARIA on custom components, reduced-motion respected) without waiting to be asked.
3. Use the screenshot → self-critique loop (Playwright, where available) before calling UI work
   done, when the environment supports it.
4. Run a quick Nielsen-heuristics self-review pass before shipping a new UI surface.

## Alternatives considered

- **Leave it as something re-derived from `dev-toolkit/research/` each time** — rejected: same
  failure mode the original global-`CLAUDE.md` decision fixed. Nothing forces a re-read.
- **Fold into the existing "Token efficiency" section** — rejected: different axis entirely (design
  quality, not cost/token tradeoffs); conflating them would muddy both.

## Reasoning

Installing `frontend-design`/`impeccable`/`modern-web-guidance` only helps if they're actually
reached for. A plugin sitting installed-but-unused is no different from not having installed it —
the standing-instruction entry is what makes the install actually change future output.
