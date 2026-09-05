# Weekly research — 2026-09-05

**Deep-dive topic:** MCP servers & integrations (rotation wraps back to 1st topic, after MCP
servers & integrations → agent architectures & multi-agent patterns → prompting techniques →
memory systems → testing/verification approaches → tooling/infrastructure/automation)

## Skills scan

Re-added all five known marketplaces (ephemeral sandbox, per standing note in `baseline.md`):
`claude-plugins-official` (unchanged at 291), `superpowers-dev` (unchanged, 1 plugin),
`claude-code-workflows` (grew 93 → 94), `impeccable` (unchanged, 1 plugin), `thedotmack`
(unchanged, 2 plugins).

The one new `claude-code-workflows` plugin, `superself`, was investigated in depth (a subagent read
its actual `.claude-plugin/plugin.json` and skill file rather than just the marketplace blurb): a
local, git-backed CLI for cross-session project-state tracking (goals, decisions, work units,
evidence-backed reports) — conceptually adjacent to what this very repo does by hand for Claude's
own setup, but scoped to a single project's task/decision history rather than cross-project tooling
decisions. Genuinely the best-behaved candidate seen yet in the memory/state-tracking category this
routine keeps circling back to (`pensyve`, `remember`, `claude-mem`, all rejected for opaque
always-on hooks): it ships **zero** automatic hooks and only activates via a description-matched
skill when a project explicitly opts in. Rejected anyway — not on hook footprint, but on plain
immaturity: ~2-week-old repo, 4 stars, 1-2 person team, self-labeled "early alpha." See
`skills-seen.md` for the full writeup and `decisions/` precedent (`claude-mem`, `hol-guard`) for why
"structurally better than what's already rejected" doesn't by itself clear the reputable-source
bar — logged as a revisit candidate, not ruled out.

Web search for new marketplaces found `claude-market/marketplace` (github.com/claude-market/
marketplace) — evaluated and not added: 22 stars/25 forks but only one plugin actually listed (a
plugin-scaffolding tool, not a catalog), no identifiable maintainer. Same thin-signal shape as the
already-rejected `xiaolai/claude-plugin-marketplace`. No other new marketplace or directory site
found beyond the two already on record (`claudemarketplaces.com`, `aitmpl.com`).

No plugin cleared the recommend bar this week.

## Token-efficiency / context-management check

Found one genuinely new, currently-undocumented lever: this harness's `Agent` tool supports
`subagent_type: "fork"`, which inherits the entire parent conversation (system prompt, tools,
model, full history) instead of starting cold, and reuses the parent's prompt cache on its first
request as a result — cheaper than a fresh subagent for a subtask that actually needs prior
context, and avoids the omission risk of restating that context by hand in a prompt. Not listed in
`baseline.md`'s "Agent architecture" section today (which only names the platform-provided agent
types: `claude`, `claude-code-guide`, `Explore`, `general-purpose`, `Plan`, `statusline-setup` —
`fork` is a distinct mode of the same tool, not a named type, which is plausibly why it's been
missed until a session's own tool description was read closely enough to notice it this week).

Drafted a recommendation with a required not-applicable boundary (per the standing quality-bar
rule): don't fork for independent/adversarial verification, where the point is that the checker
shouldn't see the reasoning that produced what it's checking; don't fork when the parent
conversation is long and mostly irrelevant to the subtask, since the bulk still rides along and can
dilute focus even at cache-cheap token cost. See
`decisions/2026-09-05-token-efficiency-fork-subagent.md`.

Also checked whether the July 2026 MCP protocol revision (see below) has any token-efficiency
angle — it doesn't directly; it's a server-authoring/transport concern, not a workflow-delegation
one, so it's covered only under the deep-dive topic below.

## Deep-dive: MCP servers & integrations

Three findings, none requiring action on our part but all worth recording:

1. **MCP spec revision, 2026-07-28 ("MCP 2.0" in informal coverage).** A real, substantial protocol
   revision: `sampling`, `roots`, and SSE-based server-to-client logging are deprecated in favor of
   a new elicitation mechanism using multi-round-trip requests (a `tools/call` can return an
   `InputRequiredResult` carrying `inputRequests` instead of completing, with an opaque
   `requestState` the client echoes back) — removing the need for a constantly-open bidirectional
   stream. Sampling itself is being unbundled in favor of servers/clients talking to an LLM
   provider directly rather than routing through MCP. Separately, "MCP Apps" extends the protocol
   to let tools return interactive HTML UI that renders in a sandboxed iframe in the chat — notably
   adjacent in shape to Claude's own Artifacts, though a distinct mechanism. None of this changes
   anything about how *we* use MCP (we don't author or host servers), but it's a useful freshness
   signal for future server evaluations: a server still built against pre-2026-07-28 patterns
   (long-lived SSE sessions, bundled sampling) is more likely to be unmaintained than one that's
   visibly moved to the new spec.
2. **Native `SearchMcpRegistry` tool.** This session has direct access to a tool that searches the
   claude.ai account-level connector directory by keyword (the same connector system `baseline.md`
   already documents — Supabase/Wallet/Gmail/Calendar/Drive) and returns install state, sample
   tools, and whether a match is enabled in the current chat. It doesn't connect anything itself —
   connecting still requires the user via the claude.ai UI, exactly as already documented — but
   it's a real, previously-undocumented discovery capability: instead of guessing at connector
   names or asking the user what's available, a session can search the directory directly. Minor,
   but worth a `baseline.md` update since it changes *how* the existing connector gap gets surfaced
   in future project work (a session can now proactively suggest a specific, real connector by
   name instead of only reactively noting "you'd need to connect X via the UI" after a user names
   it).
3. **This session's own GitHub MCP tools are a third, distinct mechanism from what `baseline.md`
   currently documents.** The baseline MCP section describes exactly two channels: the account-level
   connector system, and project-scoped `.mcp.json` (unused everywhere so far). This routine's own
   cloud session has a third: pre-configured, repo-scoped `mcp__github__*` tools supplied by the
   remote execution environment itself (not a connector the user attached, not a project
   `.mcp.json`), used here for the routine's own repo access. Worth documenting as its own category
   rather than folding it into "connector system," since its scoping and setup mechanism are both
   different (session/environment-level, tied to which repos this specific session was granted, not
   an account-wide toggle).

Compared against `baseline.md`: the account-level connector list itself (Supabase, Wallet, Gmail,
Calendar, Drive) is unchanged — no new connector appeared in the registry search performed this
week beyond what's already documented as available-but-unconnected (Sentry, Datadog, and many
others turned up in the sample query run, none newly relevant to a currently-tracked project).

## Candidate improvements

| Candidate | Impact | Effort | Status |
|---|---|---|---|
| Add `fork`-vs-fresh-subagent delegation rule to `~/.claude/CLAUDE.md` token-efficiency section | M | L | Drafted, `decisions/2026-09-05-token-efficiency-fork-subagent.md`, pending manual review |
| Update `baseline.md` Agent architecture section to list `fork` as an available subagent mode | L | L | Done this run (see below) |
| Update `baseline.md` MCP section: note `SearchMcpRegistry` tool and the session-level GitHub MCP mechanism as a third connector category | L | L | Done this run (see below) |
| Revisit `superself` once it has more than ~2 weeks of track record | M | L | Not yet — logged as a watch item in `skills-seen.md`, no action needed until it matures |
| When evaluating future MCP-server plugin candidates, treat visible adoption of the 2026-07-28 spec (vs. legacy long-lived-SSE/bundled-sampling patterns) as a maintenance-freshness signal | L | L | Noted here; no current candidate to apply it to |

No further action taken this week beyond the `decisions/` entry and the `baseline.md` updates
below — nothing else found this pass cleared the bar for a new recommendation.
