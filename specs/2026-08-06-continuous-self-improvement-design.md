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
  systems, testing/verification approaches, tooling/infrastructure/automation, skills, and token
  efficiency/context management specifically — the full breadth of "state of the art AI-assisted
  software development," not skills in isolation.
- A clear, safe autonomy boundary: what happens without a human in the loop vs. what waits for
  review.

## Non-goals

- Not a dotfiles/config-management system. This repo doesn't own or sync all of `~/.claude/` — it
  documents changes made there, it doesn't apply most of them structurally. The one deliberate
  exception is `~/.claude/CLAUDE.md` itself (see `decisions/2026-08-06-global-claude-md.md`) — a
  single, small, human-readable file this system maintains as its "compiled output" for behavioral
  decisions, not a general sync/symlink mechanism. `~/.claude/plugins` (skill installs) is managed
  by `claude plugin install` directly, same distinction — targeted, specific actions, not wholesale
  config ownership.
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

**Push notification status: still not explicitly confirmed either way** after 3 real runs
(2026-08-06) — the routine was instructed to attempt it, but nobody has explicitly confirmed a
notification actually arrived from one of these runs (as opposed to just noticing the commit in
`git log`). Worth deliberately checking on the next scheduled Saturday firing rather than assuming
either outcome. Fallback either way: the commit itself is a sufficient notification.

## Components

**`baseline.md`** — the "what do we currently have" reference. Read at the start of every research
pass so it isn't re-derived from scratch each time; updated whenever a pass finds it's drifted.

**Weekly research routine** does three things every single firing, not one topic per week — skill
discovery and token efficiency were both specifically called out as wanting to be *continuous*, so
neither sits in the same slow rotation as everything else, only getting attention once every seven
weeks:

1. **A quick skills-marketplace scan** (every week) — check for newly available skills across
   configured marketplaces (`claude plugin marketplace list` for what's configured, plus general
   research into whether other reputable marketplace sources exist that aren't yet added), diffed
   against `skills-seen.md` (see below) so the same plugin isn't re-evaluated every single week.
   See the carve-out below for what happens with what's found.
2. **A token-efficiency/context-management check** (every week) — is there a newly relevant
   technique (prompt caching, context editing/compaction, subagent model routing, delegation
   patterns, compression) not yet reflected in `~/.claude/CLAUDE.md`'s "Token efficiency" section?
   Most of the actual mechanism here is platform-managed (automatic prompt caching, harness-level
   auto-compaction) — this check is about workflow-level technique (how work gets delegated, which
   model a subagent uses, what gets read in full vs. targeted), not re-litigating the platform's own
   internals every week. **Quality bar, not just a cost bar**: a candidate technique only gets
   recommended if it costs nothing real in output quality — user feedback specifically flagged that
   low-quality work costs more later (rework, bugs, bad decisions) than tokens saved now. Every
   entry added to `~/.claude/CLAUDE.md` under this section must say when the technique should
   *not* apply, not just when it should — an efficiency instruction with no stated boundary is how
   this quietly degrades into "always cut corners" over time.
3. **One deep-dive topic** (rotates weekly, repeats after all are covered — revisiting old topics
   as the ecosystem changes is expected, not a bug). Framing: this list *is* "state of the art
   AI-assisted software development / autonomous coding agents," broken into the pieces that
   actually move independently of each other:
   1. MCP servers & integrations
   2. Agent architectures & multi-agent patterns
   3. Prompting techniques
   4. Memory systems
   5. Testing/verification approaches
   6. Tooling/infrastructure/automation

Each firing: read `baseline.md` + `~/.claude/CLAUDE.md` + the last few `research/` entries (avoid
repeating a deep-dive topic covered last week), do the skills scan, do the token-efficiency check,
do the week's deep-dive topic, compare all three against baseline, write
`research/YYYY-MM-DD-<topic>.md` covering what was found and candidate improvements rated by rough
impact/effort. Update `baseline.md` if it's gone stale. Commit and push. Send a short push
notification summarizing the finding(s) so it isn't silently invisible between sessions.

**`~/.claude/CLAUDE.md`** (global, outside this repo — see
`decisions/2026-08-06-global-claude-md.md`) — the currently-active, short summary of standing
behavioral instructions, loaded into every session automatically. `dev-toolkit` is the durable
reasoning/history; this file is what actually changes future behavior. Any decision meant to affect
*how Claude operates* (not just what's installed) gets distilled into this file during the manual
review pass, same as a plugin recommendation gets an actual `claude plugin install`.

**`skills-seen.md`** — a manifest of every marketplace+plugin combination already evaluated
(installed, or explicitly rejected with a one-line reason) so "newly available" means something
concrete instead of the scan rediscovering the same handful of plugins forever. A new marketplace
being added counts as a `skills-seen.md` event too, not just individual plugins.

**Skills lane carve-out** — the one place this routine acts on its own judgment without a review
checkpoint, though as of `decisions/2026-08-06-fix-skills-autonomy-ephemeral-sandbox.md`, "acts"
means *recommends*, not *installs in-session* — the routine's cloud sandbox is ephemeral, so a
`claude plugin install` run inside it doesn't persist anywhere real:
- A newly discovered marketplace that's clearly reputable (official, or well-established/maintained)
  gets added (`claude plugin marketplace add`) so its plugins are visible to future scans — logged
  to `skills-seen.md`. Harmless to do in-session since a future run just re-adds it from research if
  the addition itself didn't persist either; low stakes either way.
- A newly discovered skill (in an already-known or newly-added marketplace) that's (a) from a
  reputable source, (b) requires no new credentials/secrets, (c) doesn't conflict with an
  already-installed skill, and (d) clearly maps to a workflow actually in use (coding, research,
  planning, testing, deployment, data work) — gets logged as **recommended** in `skills-seen.md` and
  `CHANGELOG.md`, with a `decisions/` entry drafted explaining why. The actual `claude plugin
  install` runs during the next manual review pass, in a persistent environment where it sticks.
- A skill that was evaluated and rejected gets logged to `skills-seen.md` with the reason, so it
  isn't re-evaluated next week either.
- A skill that looks outdated, or where a clearly better alternative was found, is **not**
  auto-removed — it goes into `research/` as a candidate, same as everything else, since retiring
  something already in use is more consequential than adding something new and reversible.

**Manual implementation pass** — not scheduled, folded into normal work. Review `research/` for
unimplemented candidates, prioritize by impact vs. effort, implement what clears the bar, write a
`decisions/` entry, update `CHANGELOG.md`.

## Data flow

```
weekly cron fires (fresh git checkout of dev-toolkit)
  -> read baseline.md + ~/.claude/CLAUDE.md + skills-seen.md + recent research/
  -> skills-marketplace scan (every week)
       -> diff against skills-seen.md -> only genuinely new entries
       -> new marketplace? add it -> skills-seen.md
       -> new skill, clearly beneficial? recommend (not install) -> decisions/ + CHANGELOG.md + skills-seen.md
       -> new skill, not beneficial? -> skills-seen.md (reason logged, not re-evaluated again)
  -> token-efficiency check (every week)
       -> new technique not yet in ~/.claude/CLAUDE.md? recommend -> decisions/ + CHANGELOG.md
  -> pick this week's deep-dive topic (rotation)
       -> web research -> compare vs baseline
  -> write research/YYYY-MM-DD-<topic>.md (covers all three)
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
- If a skill recommendation turns out wrong once actually installed during manual review, that's a
  normal implementation problem — fix or revert (`claude plugin uninstall`) and log the outcome;
  not a special case.
- Nothing in this system has access to Compass's (or any other project's) secrets or credentials,
  so a failure here can't cascade into project-level damage.
- **Resolved blocker (2026-08-06, same day as discovery):** the routine's first run could
  read/checkout `dev-toolkit` but couldn't push (`GitHub access is not enabled for this session —
  an org admin must connect the Claude GitHub App`) — a GitHub OAuth App access-policy approval,
  distinct from the read/repo-visibility permission already fixed. User granted the Claude GitHub
  App access to all repos on GitHub's side; the very next run pushed successfully on the first try,
  confirmed live via `git log`. Kept here as history rather than deleted in case it regresses.

## Testing / verification

There's no application code to unit-test. Verification means: after creating the scheduled
routine, run it once manually (`RemoteTrigger` `action: "run"`) and confirm it actually produces a
real commit in the expected shape (`research/` entry, `skills-seen.md` touched, `baseline.md`
touched only if genuinely stale), that it checked out the right repo, and observe whether the push
notification attempt actually reached the user — this determines whether that part of the design
is real or needs to fall back to "the commit is the notification." Confirm the routine is
registered correctly afterward (`RemoteTrigger` `action: "list"`, human-readable cron matches
weekly). This first real run **is** the verification — not a hypothetical future step.
