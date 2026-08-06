# Continuous self-improvement system — design

**Status:** Approved. **Date:** 2026-08-06.

## Context

Working on Compass (a personal finance app) surfaced a broader ask: rather than each project
rediscovering what tooling/skills/workflows are available and useful, Claude's own operating setup
should improve deliberately over time, across every project — researched regularly, compared
against what's actually installed, with decisions and reasoning kept somewhere durable enough to
revisit as the ecosystem changes. The trigger was concrete (installing the `superpowers` plugin
mid-session), but the request is general: minimize friction and maximize effectiveness on any
future project, not just this one.

## Goals

- A durable, versioned record of what Claude's setup consists of, what's been considered, and why
  decisions were made the way they were — so past reasoning can be revisited later, not just
  repeated.
- Regular (weekly), low-effort research covering the areas that actually move: MCP
  servers/integrations, agent architectures/multi-agent patterns, prompting techniques, memory
  systems, testing/verification approaches, tooling/infrastructure/automation, and skills
  specifically.
- A clear, safe autonomy boundary: what happens without a human in the loop vs. what waits for
  review.

## Non-goals

- Not a dotfiles/config-management system. This repo doesn't own or sync `~/.claude/` — it
  documents changes made there, it doesn't apply them structurally.
- Not per-message overhead. Nothing here should run, or be checked, on every conversation turn —
  only at the start of new project work, on request, or on the weekly schedule.
- Not fully autonomous infrastructure change. Anything touching credentials, permissions, or with
  real failure modes stays behind manual review, permanently — this system doesn't graduate to
  full autonomy over time without a deliberate new decision to allow it.

## Architecture

Two loosely-coupled halves:

1. **A weekly scheduled research routine** (Claude Code's native cron-based scheduled-routine
   mechanism — not `/loop`, which is for tight iteration inside one active session, not
   multi-day-gap background work). Each firing is a stateless invocation: it reads this repo for
   context, does one topic's worth of research, writes what it found, and exits.
2. **A manual review-and-implement pass**, done by Claude during ordinary work (opportunistically —
   starting a new project, or on request), that reads the accumulated backlog in `research/` and
   acts on what's worth acting on.

The two halves communicate only through this repo's files — the routine never talks to a live
session, and a live session doesn't need the routine to be running to do useful implementation
work from what's already been logged.

## Components

**`baseline.md`** — the "what do we currently have" reference. Read at the start of every research
pass so it isn't re-derived from scratch each time; updated whenever a pass finds it's drifted.

**Weekly research routine** — topic rotation (repeats after all are covered, so revisiting old
topics as the ecosystem changes is expected, not a bug):
1. MCP servers & integrations
2. Agent architectures & multi-agent patterns
3. Prompting techniques
4. Memory systems
5. Testing/verification approaches
6. Tooling/infrastructure/automation
7. Skills (see carve-out below — this lane behaves differently from the other six)

Each firing: read `baseline.md` + the last few `research/` entries (avoid repeating a topic covered
last week), pick the next topic, search for current state-of-the-art, compare against baseline,
write `research/YYYY-MM-DD-<topic>.md` covering what was found and candidate improvements rated by
rough impact/effort. Update `baseline.md` if it's gone stale. Commit and push. Send a short push
notification summarizing the finding(s) so it isn't silently invisible between sessions.

**Skills lane carve-out** — the one place this routine can change live config on its own:
- A newly discovered skill that's (a) from a reputable source (official marketplace or a
  well-maintained repo), (b) requires no new credentials/secrets, (c) doesn't conflict with an
  already-installed skill, and (d) clearly maps to a workflow actually in use (coding, research,
  planning, testing, deployment, data work) — gets installed directly (`claude plugin install`),
  logged to `CHANGELOG.md`, with a `decisions/` entry explaining why.
- A skill that looks outdated, or where a clearly better alternative was found, is **not**
  auto-removed — it goes into `research/` as a candidate, same as everything else, since retiring
  something already in use is more consequential than adding something new and reversible.

**Manual implementation pass** — not scheduled, folded into normal work. Review `research/` for
unimplemented candidates, prioritize by impact vs. effort, implement what clears the bar, write a
`decisions/` entry, update `CHANGELOG.md`.

## Data flow

```
weekly cron fires
  -> read baseline.md + recent research/
  -> pick topic (rotation)
  -> web research
  -> compare vs baseline
  -> write research/YYYY-MM-DD-<topic>.md
  -> [skills lane only] install if clearly beneficial -> decisions/ + CHANGELOG.md
  -> update baseline.md if stale
  -> commit, push, push-notify

separately, during normal work:
  Claude reads research/ backlog
  -> implements selected candidates
  -> decisions/YYYY-MM-DD-<topic>.md + CHANGELOG.md entry
  -> commit
```

## Error handling

- If a research pass can't find anything substantive for its topic that week, it still writes a
  short entry saying so (avoids silent no-ops looking like missed runs) and moves to the next
  topic next week.
- If a skill install fails or misbehaves, that's a normal implementation problem — fix or revert
  (`claude plugin uninstall`) and log the outcome; not a special case.
- Nothing in this system has access to Compass's (or any other project's) secrets or credentials,
  so a failure here can't cascade into project-level damage.

## Testing / verification

There's no application code to unit-test. Verification means: after setting up the scheduled
routine, trigger it once manually and confirm it actually produces a real commit in the expected
shape (`research/` entry, `baseline.md` touched only if genuinely stale, push notification sent).
Confirm the schedule itself is registered correctly (list scheduled routines, check the cron
expression matches "weekly"). Recheck periodically that firings are actually happening, not
silently failing.
