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

1. **A weekly scheduled research routine**, verified against Claude Code's actual scheduling
   mechanism (not assumed — see below): a fully isolated cloud session (not `/loop`, which is for
   tight iteration inside one active local session, not multi-day-gap background work), created
   via `RemoteTrigger`/the `schedule` skill, on a weekly cron. Each firing gets a **fresh git
   checkout of this repo** (`sources: [{"git_repository": {"url":
   "https://github.com/nihadatakishiyev/dev-toolkit"}}]`) — not Compass, which is what the
   scheduling flow defaults to if not overridden — with `allowed_tools` including `WebSearch`
   (needed for the research itself, not present in the flow's generic example) alongside
   `Bash`/`Read`/`Write`/`Edit`/`Glob`/`Grep`. No MCP connectors attached (Wallet/Supabase/Gmail
   are available to attach but this routine needs none of them — attaching unused access is
   exactly the kind of unnecessary complexity this system is supposed to avoid, not add).
2. **A manual review-and-implement pass**, done by Claude during ordinary work (opportunistically —
   starting a new project, or on request), that reads the accumulated backlog in `research/` and
   acts on what's worth acting on.

The two halves communicate only through this repo's files — the routine never talks to a live
session, and a live session doesn't need the routine to be running to do useful implementation
work from what's already been logged.

**Push notification status: unverified pending first real run.** Whether a scheduled routine
session can call the notification tool the way an interactive session can isn't confirmed by the
scheduling mechanism's own documentation — the plan is to try it on the first real/test firing and
fall back to "the commit itself is the notification" (visible via `git log` / GitHub) if it turns
out not to be available in that execution context.

## Components

**`baseline.md`** — the "what do we currently have" reference. Read at the start of every research
pass so it isn't re-derived from scratch each time; updated whenever a pass finds it's drifted.

**Weekly research routine** does two things every single firing, not one topic per week — skill
discovery was specifically called out as wanting to be *continuous*, so it doesn't sit in the same
slow rotation as everything else and only get attention once every seven weeks:

1. **A quick skills-marketplace scan** (every week) — check for newly available skills across
   configured marketplaces (`claude plugin marketplace list` for what's configured, plus general
   research into whether other reputable marketplace sources exist that aren't yet added), diffed
   against `skills-seen.md` (see below) so the same plugin isn't re-evaluated every single week.
   See the carve-out below for what happens with what's found.
2. **One deep-dive topic** (rotates weekly, repeats after all are covered — revisiting old topics
   as the ecosystem changes is expected, not a bug):
   1. MCP servers & integrations
   2. Agent architectures & multi-agent patterns
   3. Prompting techniques
   4. Memory systems
   5. Testing/verification approaches
   6. Tooling/infrastructure/automation

Each firing: read `baseline.md` + the last few `research/` entries (avoid repeating a deep-dive
topic covered last week), do the skills scan, do the week's deep-dive topic, compare both against
baseline, write `research/YYYY-MM-DD-<topic>.md` covering what was found and candidate
improvements rated by rough impact/effort. Update `baseline.md` if it's gone stale. Commit and
push. Send a short push notification summarizing the finding(s) so it isn't silently invisible
between sessions.

**`skills-seen.md`** — a manifest of every marketplace+plugin combination already evaluated
(installed, or explicitly rejected with a one-line reason) so "newly available" means something
concrete instead of the scan rediscovering the same handful of plugins forever. A new marketplace
being added counts as a `skills-seen.md` event too, not just individual plugins.

**Skills lane carve-out** — the one place this routine can change live config on its own:
- A newly discovered marketplace that's clearly reputable (official, or well-established/maintained)
  gets added (`claude plugin marketplace add`) so its plugins are visible to future scans — logged
  to `skills-seen.md`, not by itself installing anything.
- A newly discovered skill (in an already-known or newly-added marketplace) that's (a) from a
  reputable source, (b) requires no new credentials/secrets, (c) doesn't conflict with an
  already-installed skill, and (d) clearly maps to a workflow actually in use (coding, research,
  planning, testing, deployment, data work) — gets installed directly (`claude plugin install`),
  logged to `CHANGELOG.md` + `skills-seen.md`, with a `decisions/` entry explaining why.
- A skill that was evaluated and rejected also gets logged to `skills-seen.md` with the reason, so
  it isn't re-evaluated next week either — same treatment as an install, minus the actual install.
- A skill that looks outdated, or where a clearly better alternative was found, is **not**
  auto-removed — it goes into `research/` as a candidate, same as everything else, since retiring
  something already in use is more consequential than adding something new and reversible.

**Manual implementation pass** — not scheduled, folded into normal work. Review `research/` for
unimplemented candidates, prioritize by impact vs. effort, implement what clears the bar, write a
`decisions/` entry, update `CHANGELOG.md`.

## Data flow

```
weekly cron fires (fresh git checkout of dev-toolkit)
  -> read baseline.md + skills-seen.md + recent research/
  -> skills-marketplace scan (every week)
       -> diff against skills-seen.md -> only genuinely new entries
       -> new marketplace? add it -> skills-seen.md
       -> new skill, clearly beneficial? install -> decisions/ + CHANGELOG.md + skills-seen.md
       -> new skill, not beneficial? -> skills-seen.md (reason logged, not re-evaluated again)
  -> pick this week's deep-dive topic (rotation)
       -> web research -> compare vs baseline
  -> write research/YYYY-MM-DD-<topic>.md (covers both)
  -> update baseline.md if stale
  -> commit, push, attempt push-notify (see note above)

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

There's no application code to unit-test. Verification means: after creating the scheduled
routine, run it once manually (`RemoteTrigger` `action: "run"`) and confirm it actually produces a
real commit in the expected shape (`research/` entry, `skills-seen.md` touched, `baseline.md`
touched only if genuinely stale), that it checked out the right repo, and observe whether the push
notification attempt actually reached the user — this determines whether that part of the design
is real or needs to fall back to "the commit is the notification." Confirm the routine is
registered correctly afterward (`RemoteTrigger` `action: "list"`, human-readable cron matches
weekly). This first real run **is** the verification — not a hypothetical future step.
