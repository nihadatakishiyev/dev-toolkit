# dev-toolkit

The paper trail for how Claude's own setup (skills, MCP servers, config, workflows) evolves over
time, across every project — not tied to any one client repo.

This repo is **passive**: it records research and decisions. It does not manage the live config
itself (that stays in `~/.claude/`, applied directly when something's implemented). Nothing here
is symlinked or synced — it's documentation and history, not runtime source of truth.

## Layout

- `baseline.md` — living snapshot of the current setup (skills, MCP servers, global config). Each
  research pass diffs against this instead of re-deriving it from scratch.
- `specs/YYYY-MM-DD-<topic>.md` — full design docs for anything substantial enough to need one
  (like this system's own design).
- `research/YYYY-MM-DD-<topic>.md` — dated notes from each scheduled research pass: what was
  found, how it compares to `baseline.md`, candidate improvements with impact/effort. New skills
  that are clearly beneficial get installed directly during the pass (see spec) rather than only
  logged here.
- `decisions/YYYY-MM-DD-<topic>.md` — ADR-style record for anything actually implemented: context,
  decision, alternatives considered, reasoning. Written when work lands, not when it's proposed.
- `CHANGELOG.md` — running index of what actually changed, newest first, linking to the fuller
  decision record for each entry.

## How it's used

1. A weekly scheduled routine researches one rotating topic, writes a `research/` entry, updates
   `baseline.md` if it's gone stale, and auto-installs any clearly-beneficial newly-discovered
   skill (logged to `CHANGELOG.md` + `decisions/`). Everything else it finds just becomes a
   candidate in the backlog.
2. Claude reviews the `research/` backlog opportunistically — starting new project work, or when
   asked — and implements what clears the impact/effort bar, logging each as it lands.

See `specs/2026-08-06-continuous-self-improvement-design.md` for the full design and reasoning.
