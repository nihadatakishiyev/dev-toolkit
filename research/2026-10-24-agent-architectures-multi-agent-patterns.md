# Weekly research — 2026-10-24: Agent architectures & multi-agent patterns

Fourteenth weekly research pass. Rotation: MCP servers & integrations (2026-10-17) → **Agent
architectures & multi-agent patterns** (this run, lap 3's 2nd topic) → Prompting techniques next.

## Skills scan

**The headline finding this week is structural, not an individual plugin.** The built-in plugin
marketplace has been renamed and grown roughly 11x:

- `claude plugin marketplace list` no longer shows `claude-plugins-official` by that name at all.
  The only built-in entry is now **`anthropic-plugin-directory`** ("Built in (Anthropic
  Directory)").
- `claude plugin list --available --json` shows it carrying **3,598 plugins**, up from 315 as
  recently as 2026-10-17 — not a gradual drift, an order-of-magnitude jump.
- Two independent first-party sources corroborate why: claude.ai's own support article "Browse
  skills, connectors, and plugins in one directory" describes a single unified directory for
  skills/connectors/plugins on claude.ai (updated more than a month before this fetch, no exact
  date given); and Claude Code's changelog (v2.1.299, 2.1.304, 2.1.309 — Oct 14–22, 2026) describes
  plugins/skills now **syncing from claude.ai into Claude Code sessions**, explicitly including
  Cowork cloud sessions (the execution context this very routine runs in), using the terms
  "claude.ai Directory plugin," "account plugin," and "claude.ai-synced plugin."
- This plausibly supersedes/collapses two of the three catalogs the 2026-10-17 pass documented as
  separate: claude.ai's account-level plugin catalog (previously empty) and its personal Skills
  library (previously 14 entries) look like they're now surfacing through this one CLI-visible
  marketplace instead of staying siloed.
- **Trust-model consequence:** spot-checking three names pulled from it —
  `agentic-council`→`github.com/dtsong/agentic-council`, `agent-protocols`→
  `github.com/arneesh/agent-protocols`, `skills-for-real-engineers`→`github.com/mattpocock/skills`
  — confirms it hosts arbitrary third-party GitHub repos, not just Anthropic-authored plugins.
  Baseline previously treated "claude-plugins-official" as the tightly-curated default; that
  assumption no longer holds for its replacement. It needs the same per-plugin vetting discipline
  already applied to `claude-community`, not a lighter touch just because it's "built in."
- **Not yet independently reconfirmed from a persistent (non-ephemeral) environment.** This is a
  single ephemeral-sandbox observation, corroborated by official docs/changelog text but not by a
  second live session. Treat as strong-but-unconfirmed, same caution baseline already applies
  elsewhere to sandbox-specific findings.
- Two loose threads, not acted on: `chrome-devtools-mcp` and `modern-web-guidance` (the latter
  actually installed in the persistent environment per baseline) weren't found by exact name in
  this new directory snapshot — could be a rename, could be a sync-timing artifact, not treated as
  a confirmed removal. Separately, `superpowers` shows v7.0.1 available here vs. 6.2.0 in the
  documented persistent install — a version bump worth picking up at next manual review, not a new
  evaluation.

Six marketplaces re-added this run (ephemeral sandbox, as always): `claude-code-workflows`
unchanged at 94; `impeccable`/`superpowers-dev`/`thedotmack` unchanged (1/1/2 plugins);
`claude-community` unchanged at 2,283 (flat after last week's first-ever shrink). This week's
deep-dive-topic keyword sweep (`agent`/`orchestrat`/`multi-agent`/`swarm`/`team`) against
`claude-community` matched 149/2,283 entries (`ag2-claude-plugins`, `agent-discussion`,
`ai-team-os`, `atelier`, `attacca`, `building-multiagent-systems`, `cc10x-router-owned-harness`,
dozens more) — far too many to vet individually in a weekly pass, same precedent as the
2026-10-10/2026-10-17 large-marketplace sweeps. The same sweep against `claude-code-workflows`
matched only 8 names, all already evaluated/rejected in earlier passes — confirms zero new growth
there. Individually evaluated from the restructured `anthropic-plugin-directory` (see
`skills-seen.md`): `agentic-council` (20-specialist council — overlaps the native `Workflow`
tool's judge-panel idiom, rejected), `agent-protocols` (overlaps superpowers' plan/execute/TDD
pipeline, rejected), `skills-for-real-engineers` (same Matt Pocock repo already rejected
2026-08-15 under a different marketplace listing, same verdict). Web search for other
not-yet-configured marketplaces found nothing new beyond the already-known discovery-directory
sites (`claudemarketplaces.com`, `aitmpl.com`), plus one more of the same kind (`vibeindex.ai`) —
not an installable marketplace itself.

## Token-efficiency / context-management check

No new technique found this week that clears the quality bar as a `~/.claude/CLAUDE.md`
recommendation. Two mechanism-level facts surfaced while mining the changelog for the deep-dive
topic, noted here for completeness rather than drafted as recommendations:

- **`CLAUDE_CODE_WORKFLOW_SUBAGENT_MODEL`** (v2.1.296, Oct 9, 2026) — an env var that forces every
  agent spawned by the `Workflow` tool onto one model, while ordinary subagents keep their own.
  Not drafted as a recommendation: it's a blunt global override (one model for *every* workflow
  agent, not per-task routing), and the `Workflow` tool itself is still "documented, zero
  real-task usage" per baseline — there's no concrete scenario yet to safely recommend a specific
  setting without risking exactly the "always cut corners" failure mode the quality bar exists to
  prevent.
- **`subagent_type: "fork"` is now on by default more broadly** (v2.1.302, Oct 16, 2026) — in
  `claude -p`, the Agent SDK, the desktop app, and VS Code, joining the terminal where it was
  already default (`CLAUDE_CODE_FORK_SUBAGENT=0` opts out). This changes *availability*, not the
  existing cost-tradeoff guidance in `decisions/2026-09-05-token-efficiency-fork-subagent.md`
  (fork is cheaper when a subtask genuinely needs prior context, worse for independent/adversarial
  verification or when the parent conversation is mostly irrelevant bulk) — that reasoning still
  holds unchanged, just reaches more surfaces now. Added to baseline's Agent architecture section
  as a fact, no new decision needed.

General web search (auto-compact triggers, manual `/compact` timing, third-party context-pruning
tools) surfaced nothing new or official dated this week — same guidance already in circulation,
confirming pass.

## Deep-dive: Agent architectures & multi-agent patterns

Compared against baseline's existing section (the five-pattern Anthropic taxonomy from
2026-09-12: Orchestrator-Subagent, Generator-Verifier, Agent Teams, Message Bus, Shared State; the
`fork` subagent mode; the `Agent` tool's `effort` parameter found 2026-10-10).

**What changed, mined directly from the Claude Code changelog (v2.1.288 through v2.1.310,
Oct 2–23, 2026) rather than third-party blog claims:**

- `subagent_type: "fork"` default-on expansion — see above.
- **Agent Teams remains experimental, not GA**, for the third time this gets explicitly checked
  (2026-08-06 original, 2026-09-12 reconfirmed, now 2026-10-24). This pass used a stronger evidence
  bar than before: direct changelog mining rather than web search, which this week turned up only
  third-party blogs disagreeing with each other about GA status (one claiming a February 2026
  launch "shipped," others still calling it experimental as late as June 2026). The official
  changelog shows active, ongoing bug fixes through October — a plugin-defined agent spawned by
  name not inheriting its own prompt/tools/effort (2.1.288), approving a teammate shutdown request
  exiting the whole CLI (2.1.304, fixed), split-pane teammates not inheriting the leader's
  tool-deny rules (2.1.309, fixed) — consistent with "still rough, still being hardened," not
  consistent with a quiet graduation to GA. No entry anywhere in this window uses "GA," "generally
  available," or removes the experimental-flag language. Baseline's "not GA, documented rough
  edges" note is reconfirmed, not superseded.
- **Mod hooks are now explicitly agent/teammate-aware**: `turn.start` now fires for subagents' and
  teammates' turns too (previously main-loop-only), and a new `agent.spawn` hook fires when a
  finished subagent or killed teammate is resumed, and can refuse the resume (both v2.1.299). Ties
  to baseline's "Claude Mods" entry — still no identified use case for this account, noted as a
  capability fact only.
- **Workflow tool**, several small fixes (CRLF script refusal 2.1.296, clearer `scriptPath` error
  2.1.295, resume-via-approval-instead-of-handoff-file for background sessions 2.1.300) — routine
  hardening, nothing that changes how/when to reach for it.
- Tangential, not Claude Code itself: the Claude API's own **Managed Agents** feature (mentioned in
  passing by the bundled `claude-api` skill's description) moved its default subagent-team model to
  `multiagent_20261001` with "dynamic workflows on by default" (v2.1.309). No tracked project builds
  directly on the raw Claude API/Agent SDK (same precedent as the already-rejected `agent-sdk-dev`
  plugin) — noted for completeness, not actionable here.

**What the broader ecosystem is saying** (web search, mostly March–June 2026 sources — nothing
dated October 2026 was found, a genuine gap rather than a missed search): the field's converged
view hasn't moved from what baseline already reflects. The most mature production pattern remains
a single primary agent spawning focused subagents, not full peer-to-peer multi-agent coordination;
multi-agent coordination is specifically flagged as adding overhead that often isn't worth paying
until a single-agent approach hits a clear wall. One piece of vocabulary worth recording (not a
behavior change, same spirit as recording "loop engineering" on 2026-09-19): **"confidence-gated
escalation"** — start with a cheap path (cheaper model, fewer agents) and escalate only when a
confidence signal calls for it. This is a more precise name for a shape this setup's existing
Haiku-routing + `effort`-parameter practice already approximates, not a new technique to adopt.

**Cross-reference to the standing testing/verification gap**: the `Workflow` tool's
Generator-Verifier/judge-panel idiom is now on its **fifth consecutive deep-dive mention**
(2026-08-22 original flag, reinforced 2026-09-12/2026-09-19/2026-10-03, now again) as "documented,
available, zero real-task usage." This routine can't force that trial itself — it's a manual-review
job — but the streak is long enough that it's worth stating plainly rather than softening: whatever
is keeping this idiom from ever getting a real try is worth asking about directly next time a
verification-shaped task comes up, rather than filing one more "still not exercised" note.

## Candidate improvements

| Candidate | Impact | Effort |
|---|---|---|
| Reconfirm the `anthropic-plugin-directory` rename/11x-growth finding from a persistent (non-ephemeral) environment | H | L |
| Update baseline's skills/MCP section for the `anthropic-plugin-directory` rename and its weaker (third-party-hosting) trust model — done this pass, see below | M | L |
| Note `subagent_type: "fork"`'s broader default-on reach (v2.1.302) in baseline's Agent architecture section — done this pass | L | L |
| Actually exercise the `Workflow` tool's Generator-Verifier/judge-panel idiom on a real verification-shaped task (5th consecutive "unused" flag) | M | M |
| Reconfirm whether `chrome-devtools-mcp`/`modern-web-guidance` are still present/installed in the persistent environment (name mismatch observed only in the ephemeral sandbox this run) | L | L |
| Sync the `superpowers` version bump (6.2.0 → 7.0.1 available) at next manual review and skim its changelog for anything behavior-relevant | L | L |
