# Weekly research — 2026-08-06

**Deep-dive topic:** Agent architectures & multi-agent patterns (2nd in rotation, after MCP servers
& integrations)

## Skills scan

Ran the scan against all three previously-known marketplaces (re-added this run since the routine's
sandbox is ephemeral and nothing persists between runs — see `baseline.md`'s known blocker), and
found one new marketplace via web search.

**New marketplace found and added:** `anthropics/claude-code` (`claude-code-plugins`) — a
first-party Anthropic marketplace hosted directly in the `claude-code` repo itself, distinct from
`claude-plugins-official`. Turned out to be a 13-plugin subset that's already fully reachable
through `claude-plugins-official`'s much larger 278-plugin listing, with one exception
(`claude-opus-4-5-migration`) that's moot since this account is already on the Claude 5 model
family. No unique actionable plugins — see `skills-seen.md` for the full note.

**Also searched for other marketplaces not yet configured**, per the routine's every-week mandate:
found `claudemarketplaces.com` (a curated *directory* of marketplaces/plugins, not itself an
installable source — worth using as a discovery aid in future scans) and
`xiaolai/claude-plugin-marketplace` (a single-maintainer personal marketplace with no visible
reputability signal — not added).

**First real per-plugin dive into `claude-plugins-official`'s 278 plugins** (previously only logged
as "known," never evaluated plugin-by-plugin) and a deeper look at `wshobson/agents`'s now-95
plugins (up from ~94 last week — one or more added upstream since). Full results are in
`skills-seen.md`; summary:

- **3 recommended** (drafted `decisions/` entries, not installed — this run's sandbox is
  ephemeral): `playwright` (Microsoft's browser-automation MCP server — directly closes the
  browser E2E testing gap `baseline.md` already names), `block-no-verify` (a `PreToolUse` hook that
  enforces the no-`--no-verify`/no-bypass-flags rule this harness's own instructions already state,
  turning a prompt-level rule into an enforced one), and `claude-code-setup` (first-party
  Anthropic plugin that automates the "what does this new project need" analysis this repo's own
  manual-review pass already does by hand).
- **~24 rejected**, each with a one-line reason in `skills-seen.md` — most either overlap with
  something already installed (superpowers' planning/orchestration/TDD/review skills, the bundled
  `simplify`/`review`/`security-review` skills), don't clearly map to an active workflow yet
  (QA-agent toolkits, agent-regression-testing, product-risk review), or introduce complexity this
  routine's carve-out treats as a soft disqualifier even when not a literal "credential" (crypto
  key material for policy-signing plugins).
- `pensyve` (memory runtime, previously rejected for auto-running hooks with no independent
  security review) got an echo this week: `remember`, a similarly-scoped memory plugin that's
  *better* documented (explicit local-only-by-default security model, reuses Claude Code's own
  OAuth token) but ships the same shape of automatic-hook risk. Rejected for consistency with the
  `pensyve` call rather than judged more leniently — flagged to revisit both together, not
  separately, if either gets more track record or outside review.

No marketplace or plugin previously logged in `skills-seen.md` needed re-evaluation — the diff
worked as designed.

## Deep-dive: Agent architectures & multi-agent patterns

- **Three-tier landscape, as of mid-2026**: (1) *subagents* — spawned within one session, report a
  summary back to the main agent, never talk to each other or share state; cheap because only the
  summary re-enters context. (2) *Agent Teams* — a newer Claude Code feature (shipped February
  2026 alongside Opus 4.6) where multiple full Claude Code instances share a task list with a
  lock-and-claim mechanism, message each other directly, and one instance acts as orchestrator.
  Still **experimental and disabled by default**, gated behind
  `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`, with documented known limitations around session
  resumption, task coordination, and shutdown behavior. Each teammate is a full instance, so a team
  costs meaningfully more tokens than a single session or a subagent fan-out. (3) *external
  orchestrators* — tools that coordinate Claude Code sessions across machines/branches/repos,
  adding dashboards and git-worktree tracking the built-in features don't cover.
- **Named orchestration patterns** circulating in 2026 guidance: fan-out, pipeline, debate,
  supervisor, and swarm. The recurring advice across sources is to match pattern to task shape
  first and pick tooling second — swarm-style setups on tasks a simple fan-out could handle is
  called out repeatedly as wasted engineering/token budget, not a sign of sophistication.
- **This account's actual capability, compared against the above**: this session has direct access
  to an `Agent` tool (subagent tier — named agent types: `claude`, `claude-code-guide`, `Explore`,
  `general-purpose`, `Plan`, `statusline-setup`) *and* a `Workflow` tool that implements
  deterministic multi-agent orchestration in code — `pipeline()`/`parallel()` primitives, phases,
  schema-validated structured output, adversarial-verify and judge-panel patterns as documented
  idioms. Superpowers' `dispatching-parallel-agents` skill sits at the same subagent tier. None of
  this — the `Agent`/`Workflow` tools, or Agent Teams' existence and experimental status — was
  previously written down anywhere in `baseline.md`. That's a real documentation gap this pass
  surfaced, independent of anything found externally: **the account already has meaningfully more
  multi-agent capability available than `baseline.md` currently describes.**
- Agent Teams specifically is not something to turn on based on this research alone — it's
  experimental, off by default, with known rough edges, and no identified task in current project
  work (Compass, dev-toolkit) that needs full-instance teammates rather than subagent fan-out or
  the `Workflow` tool's deterministic pipelines. Worth revisiting once it graduates out of
  experimental status.

## Compared against `baseline.md`

Baseline currently has no "agent architecture" section at all — it documents skills, MCP
connectors, and global config, but nothing about the multi-agent tooling already available in this
very session (`Agent`, `Workflow`) or the platform-level Agent Teams feature. This is stale by
omission rather than by an outdated fact, so this pass updates `baseline.md` with a new section
describing what's actually available today, at the tier each capability sits at (subagent vs.
full-instance team vs. external), and Agent Teams' experimental/opt-in status specifically so a
future pass doesn't mistake "not mentioned" for "not available."

## Candidate improvements

| Candidate | Impact | Effort |
|---|---|---|
| Document the account's actual `Agent`/`Workflow` tool capability in `baseline.md` (done this pass) | M | L |
| Revisit `pensyve`/`remember` together once either has more track record or a visible independent security review of their hooks | M | L |
| Try Agent Teams (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) once it's out of experimental status, on a task that genuinely needs full-instance teammates messaging each other rather than subagent fan-out | L | M |
| Use `claudemarketplaces.com` as a discovery aid in future skills scans, not just direct web search | L | L |
| Revisit `qa-orchestra` / `ciagent` (multi-agent QA and agent-regression-testing toolkits) if a project starts doing heavy browser QA or ships its own AI agent that needs regression testing | L | M |
