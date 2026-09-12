# Weekly research — 2026-09-12

**Deep-dive topic:** Agent architectures & multi-agent patterns (rotation's 2nd topic, revisited
after MCP servers & integrations → agent architectures → prompting techniques → memory systems →
testing/verification approaches → tooling/infrastructure/automation → MCP servers & integrations
again on 2026-09-05; this is the first *re-visit* of this specific topic since its first pass on
2026-08-06)

## Skills scan

Re-added all five known marketplaces (ephemeral sandbox, per standing note in `baseline.md`):
`claude-plugins-official` (**grew 291 → 295**), `superpowers-dev` (unchanged, 1 plugin),
`claude-code-workflows` (unchanged at 94, matches the count already recorded 2026-09-05),
`impeccable` (unchanged, 1 plugin), `thedotmack` (unchanged, 2 plugins).

**New marketplace found and added: `anthropics/claude-plugins-community` (`claude-community`).**
Web search surfaced this — a first-party Anthropic-run community marketplace, distinct from
`claude-plugins-official`, that syncs nightly from an internal review pipeline: anyone submits a
plugin via `clau.de/plugin-directory-submission`, it passes automated security scanning, and gets
added. Added per the standing rule (official host = reputable). Its actual listing is **2,282
plugins** — far larger than anything else configured. Important nuance for future scans, not
previously distinguished in this file: "official host" and "reputable plugin" are different bars.
`claude-plugins-official`'s plugins are individually curated by Anthropic for quality;
`claude-community`'s plugins only pass an automated *security* scan (checking for malicious code),
which says nothing about an individual submitter's track record, maintenance, or fitness for a
given workflow — the same reputable-source bar this routine applies to any third-party plugin
still applies per-plugin here, the host being official doesn't waive it. At 2,282 entries, keyword
search surfaced hundreds of agent-orchestration-themed plugins (`atelier`, `claude-alloy`,
`attacca`, `building-multiagent-systems`, `agent-tasks`, and many more) directly relevant to this
week's deep-dive topic, but none were individually vetted for reputability this run — no
star/maintainer metadata is visible in the marketplace listing itself, and vetting hundreds of
unknown single-submitter plugins in one low-effort pass isn't feasible. Treating this the same way
the routine already treats `wshobson/agents`'s and `claude-plugins-official`'s own long tails: no
plugin recommended from it this week; future weeks should do narrow, deep-dive-topic-driven keyword
searches (as attempted here) rather than a full re-scan, and only individually evaluate one if it's
independently cited by name in a reputable outside source, or a narrow search turns up something
that looks unusually well-established (visible org backing, high fork count relative to stars,
etc.) on closer inspection.

**Second candidate marketplace found, not added:** `davepoon/buildwithclaude`
(github.com/davepoon/buildwithclaude, buildwithclaude.com) — 3.4k★/502 forks, single maintainer
(Dave Poon). Higher star count than any previously-rejected discovery-site candidate, but it's
fundamentally the same shape as the already-rejected `tonsofskills.com`/
`claude-code-plugins-plus-skills` (indexes 20k+ community plugins/4,500+ MCP servers from a single
maintainer with no independent per-entry vetting) — same infeasible-vetting-at-scale reasoning
applies despite the better star count. Not added.

**`claude-plugins-official`'s +4 growth (291 → 295):** the local marketplace clone is a shallow
depth-1 checkout with no usable git history, so the exact new plugin names couldn't be diffed
precisely against last week's snapshot (no full name list was persisted anywhere to diff against —
only counts were recorded). Given this week's deep-dive topic, did a targeted individual evaluation
of names in the current listing that map to agent-architecture themes and hadn't been individually
evaluated before (same targeted-keyword approach used on 2026-08-22/2026-08-29 for the
`claude-code-workflows` bulk tail, applied here to `claude-plugins-official` instead):

| Plugin | Verdict | Reason |
|---|---|---|
| `atomic-agents` | rejected | Framework-specific dev workflow for the Atomic Agents Python framework — no project here uses it. |
| `agent-sdk-dev` | rejected | Dev kit for building apps with the Claude Agent SDK directly — no project currently builds a standalone Agent-SDK-based app (this repo/Compass both operate through Claude Code itself, not the SDK). Revisit if that changes. |
| `pydantic-ai` | rejected | Framework-specific guidance for Pydantic AI — no project uses it. |
| `forge-skills` | rejected | Atlassian Forge app scaffolding/MCP — no Forge project. |
| `ai-plugins` (Endor Labs) | rejected | Requires an Endor Labs account/`endorctl` setup — new-credential disqualifier. |
| `cwc-makers` | rejected | Onboards a physical "Code-with-Claude Makers Cardputer" hardware device — no mapped workflow. |

None of the six is a loss — all fail the "no mapped current workflow" or "new credential" bar
already established, consistent with how the rest of `claude-plugins-official`'s long tail has been
handled. No plugin cleared the recommend bar this week from any marketplace.

## Token-efficiency / context-management check

Checked specifically for anything new since the `fork`-subagent finding logged 2026-09-05. Two
things turned up in web search, neither actionable:

- Prompt caching / Haiku-tiering / bounded-context best-practice writeups — all describe exactly
  the levers already reflected in `~/.claude/CLAUDE.md` (Haiku routing, targeted reads over
  whole-file reads) and `baseline.md` (`fork` mode). Confirming, not new.
- Anthropic's Messages API **context editing** (`context-management-2025-06-27` beta) and
  **memory tool** (`memory_20250818`) — real, substantive features (reported 29–39% performance
  improvement, up to 84% token savings on long tool-use-heavy tasks in Anthropic's own benchmark)
  but they're API-level primitives for *building* a custom agent against the raw Messages API, not
  something that changes how Claude Code itself (this session, or any session doing work in a
  tracked project) delegates or manages its own context — that's already automatic/harness-managed,
  per this repo's own non-goal ("not re-litigating the platform's own internals every week").
  Relevant only if a tracked project starts building its *own* Claude-API-based agent (e.g. a
  Compass Edge Function) — that would be a project-level engineering decision for that project, not
  a `~/.claude/CLAUDE.md` behavioral rule for this account. Noting here so it doesn't get
  rediscovered as new next time; not recommended for `~/.claude/CLAUDE.md`.

No new technique recommended this week — nothing found meets both "not yet reflected" and
"actually changes how Claude Code's own delegation/workflow behavior should work" at once.

## Deep-dive: Agent architectures & multi-agent patterns

`baseline.md`'s existing three-tier structure (Subagents incl. `fork` mode; `Workflow` tool;
Agent Teams) was checked against Anthropic's own current public framing, and holds up well:

1. **Anthropic published (or at least, this is the first time this routine surfaced) two directly
   relevant posts: "When to use multi-agent systems (and when not to)" and "Multi-agent
   coordination patterns: Five approaches and when to use them"** (both `claude.com/blog/...` —
   `claude.com` is blocked by this session's egress proxy, so read via search-result summaries and
   secondary sources rather than fetched directly; the substance below is corroborated across
   multiple independent summaries, not single-sourced). Headline guidance: multi-agent systems are
   frequently reached for where a single, better-prompted agent would do better — start with the
   simplest pattern that could work and evolve only where it visibly struggles. Tasks with
   tightly-coupled state, sequential dependencies, or shared mutable context hit coordination
   overhead faster than they gain from parallelism.
2. **The five named coordination patterns map cleanly onto what's already documented here:**
   - *Orchestrator-Subagent* (lead agent plans/delegates/synthesizes, subagents handle bounded
     tasks) — Anthropic's own stated default/recommended-starting-point pattern, "handles the
     widest range of problems with the least coordination overhead." This is exactly this
     session's own `Agent` tool usage pattern, already the primary mechanism in use here.
   - *Generator-Verifier* (one agent produces, a separate agent checks against explicit criteria)
     — exactly the adversarial-verify idiom `baseline.md`'s Testing/verification section already
     names as available via the `Workflow` tool, and already flagged there as "documented, zero
     real-task usage" for two consecutive testing-topic passes now (2026-08-22, and implicitly
     still true). Anthropic's own writeup reinforces this is a real, named, recommended pattern —
     not a niche idea — which strengthens rather than changes the existing "try this for real" flag.
   - *Agent Teams* — matches `baseline.md`'s existing entry exactly; independently confirmed via
     web search still not GA ("has not yet reached full general availability... research preview"
     as of this search), consistent with the experimental-flag status already documented. No
     baseline update needed here, just reconfirmation.
   - *Message Bus* (event-driven, growing/pluggable agent ecosystem) and *Shared State*
     (blackboard-style collaborative state) — **no current analog in this account's tooling, and no
     identified project need for either.** Worth naming as a known gap for completeness (this
     topic's coverage was previously three tiers; Anthropic's own taxonomy has five), but not an
     action item — nothing tracked here has an "event-driven pipeline of many pluggable agents" or
     "several agents collaborating over literally shared mutable state" shape yet.
3. **Net assessment: this is a confirming pass, not a gap-closing one**, similar in character to
   the 2026-08-08 prompting-techniques pass — current three-tier practice already matches 2 of
   Anthropic's 5 named patterns as primary/default mechanisms, and knowingly doesn't use the other
   3 (Generator-Verifier available-but-unexercised, Message Bus/Shared-state not applicable to any
   current project shape). No architecture change indicated.

## Candidate improvements

| Candidate | Impact | Effort | Status |
|---|---|---|---|
| Add `claude-community` marketplace note + the "official host ≠ reputable plugin" distinction to `baseline.md` | M | L | Done this run (see below) |
| Actually exercise the `Workflow` tool's Generator-Verifier (adversarial-verify) idiom on a real task | M | M | Not yet — third consecutive research pass (this one plus the two prior testing-topic ones) flagging this same "documented, unused" gap; consider forcing this the next time a real verification-shaped task comes up, rather than letting it ride a fourth time |
| Revisit `claude-community` with narrow, topic-driven keyword searches in future weekly passes instead of a full re-scan | L | L | Noted here as the intended approach going forward |
| If a tracked project ever builds its own Claude-API-based agent, evaluate context editing/memory tool for that project specifically | L | L | Not applicable to any current project; noted so it isn't rediscovered as "new" |

No decisions/ entries drafted this week — no plugin or token-efficiency technique cleared the
recommend bar. This week's actionable content is the marketplace addition (logged directly to
`skills-seen.md`, per the standing low-stakes marketplace-add rule) and the `baseline.md` updates
below.
