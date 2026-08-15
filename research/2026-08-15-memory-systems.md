# Weekly research — 2026-08-15

**Deep-dive topic:** Memory systems (4th in rotation, after MCP servers & integrations → agent
architectures & multi-agent patterns → prompting techniques)

## Skills scan

Re-added all five previously-known marketplaces this run (`claude-plugins-official`,
`superpowers-dev`, `claude-code-workflows`, `impeccable`, `thedotmack`) — the routine's cloud
sandbox is ephemeral, so nothing persisted from the 2026-08-08 run; see `baseline.md`'s "Weekly
research routine" section for the standing note on this. Web search for other reputable
marketplaces not yet configured turned up nothing new beyond the two curated directory sites
already on record (`claudemarketplaces.com`, `aitmpl.com`).

- `superpowers-dev`, `impeccable`, `thedotmack` — unchanged (still 1 plugin each, same names).
- `claude-code-workflows` (wshobson/agents) — unchanged at 95 plugins, matching the 2026-08-08
  count. All already itemized or bulk-rejected in `skills-seen.md`; nothing new to evaluate.
- `claude-plugins-official` — grew from 284 to 286 plugins. A name-level sweep against
  `skills-seen.md` surfaced ~14 plugins not yet itemized by name there (some may be genuinely new,
  others just previously swept into the 2026-08-08 bulk-reject entry without being named — no clean
  way to tell which, given `skills-seen.md` doesn't store the full prior list). Evaluated each on
  its own merits:
  - **Recommended:** `chrome-devtools-mcp` (official Chrome DevTools team — live browser
    inspection/debugging, distinct from `playwright`'s test-automation role) and `typescript-lsp`
    (first-party TypeScript/JS language server, matches Compass's actual stack). See
    `decisions/2026-08-15-install-chrome-devtools-mcp.md` and
    `decisions/2026-08-15-install-typescript-lsp.md`.
  - **Rejected:** the rest of the per-language LSP family (no mapped current-project language),
    `serena` (redundant with `typescript-lsp` for a single-language stack), `claude-security`
    (overlaps bundled `security-review`), `coderabbit`/`greptile`/`qodo`/`sonarqube` (SaaS
    accounts/credentials), `mattpocock-skills` (overlaps superpowers), `mcp-server-dev` (no current
    need), `skill-creator` plugin (duplicate of the bundled skill), `superdesign`/`ui-theme-designer`
    (overlap existing design tooling / no mapped project), `explanatory-output-style` (not a named
    workflow category). Full reasoning for each in `skills-seen.md`.

## Token-efficiency / context-management check

No new *runtime* technique found beyond what's already tracked — prompt caching and compaction
remain platform-managed (out of scope per this check's own carve-out: not re-litigating platform
internals weekly), and nothing new turned up about subagent model/effort routing beyond the
effort-routing recommendation already drafted 2026-08-08
(`decisions/2026-08-08-token-efficiency-effort-routing.md`), which is **still open** — not yet
applied to `~/.claude/CLAUDE.md`, still waiting on a manual review pass. Flagging that it's pending,
not re-researching it.

One genuinely new item did surface, but it came out of this week's deep-dive reading rather than a
separate token-efficiency search: `.claude/rules/` path-scoped instruction files. See the deep-dive
section below and `decisions/2026-08-15-token-efficiency-path-scoped-rules.md` for the full
recommendation and its required not-applicable boundary.

## Deep-dive: memory systems

Read Claude Code's official memory docs (`code.claude.com/docs/en/memory`) and subagent docs
(`code.claude.com/docs/en/sub-agents`) directly, end to end, rather than relying on search-summary
blog content (a first search pass surfaced some imprecise claims — see "Corrections" below).

**Confirmed, matches `baseline.md`:** auto memory (Claude's own accumulated notes, distinct from
`CLAUDE.md` which the user writes) lives at `~/.claude/projects/<project>/memory/`, keyed by git
repo so all worktrees/subdirectories share one memory directory, on by default
(`autoMemoryEnabled`), with a `MEMORY.md` entrypoint (200-line/25KB load cap, Claude self-curates
when near the limit) plus on-demand topic files. Machine-local, not synced across machines. This
matches what `baseline.md` already documents — the mechanism itself hasn't changed since the
2026-08-06 baseline, though the docs are considerably more detailed than what's summarized there.

**New, not yet reflected in `baseline.md`:**

1. **Subagent-level `memory` field.** A custom subagent (defined via a `.claude/agents/*.md` file
   with YAML frontmatter — not the platform-provided named agent types like `Explore`/`Plan`/
   `general-purpose` this account currently uses) can declare `memory: user|project|local`, giving
   that specific subagent its own persistent directory (`~/.claude/agent-memory/<name>/`,
   `.claude/agent-memory/<name>/`, or `.claude/agent-memory-local/<name>/` respectively) separate
   from the main conversation's auto memory. **Critical dependency:** this only works if the
   session's own auto memory is enabled — `autoMemoryEnabled: false` or
   `CLAUDE_CODE_DISABLE_AUTO_MEMORY` silently disables the subagent's memory field too, no separate
   toggle.
   - **Not currently actionable:** no project tracked in this repo defines any custom subagents via
     `.claude/agents/*.md` — the account's available agent types are all platform-provided. This
     stays a "watch" item, not a recommendation, until a project actually has a repeatedly-invoked
     custom subagent worth giving persistent memory to (e.g. a project-specific migration-runner or
     domain-modelling subagent that would benefit from remembering schema quirks across
     invocations instead of re-deriving them from prompt context every time).
2. **`.claude/rules/` path-scoped instruction files.** Distinct from both `CLAUDE.md` and auto
   memory — markdown files in `.claude/rules/` (or `~/.claude/rules/` for user-level) that load
   into context only when Claude reads a file matching their `paths:` frontmatter glob, instead of
   loading in full at every session start. This is genuinely actionable — see the token-efficiency
   recommendation above and `decisions/2026-08-15-token-efficiency-path-scoped-rules.md`.
3. **`modified` frontmatter timestamp** (Claude Code v2.1.214+) — when Claude writes a memory file
   that already has YAML frontmatter, Claude Code stamps an ISO-8601 write time into a `modified`
   field automatically. Minor, no action needed — just a freshness signal for memory Claude already
   maintains itself.

**Separately, not applicable here:** the *Managed Agents API* (a different Anthropic product for
building custom hosted agents, not the Claude Code CLI this repo tracks) has its own, unrelated
"memory store" concept — workspace-scoped document collections mounted into a session's sandbox,
attached at session-creation time, with explicit versioning/audit/redaction and a "dreaming session"
consolidation feature. Worth naming here only because search results initially conflated the two
products' memory systems under generic "Claude memory" framing — confirmed via the official docs
that they're unrelated mechanisms serving different products. No action; this repo doesn't track
Managed Agents API usage anywhere.

**`claude-mem` (thedotmack) status:** no material change found this week beyond marketing-content
search hits; still resting on the 2026-08-08 rejection (auto-running six-hook footprint, unreviewed
worker-service source) — not re-litigated.

### Corrections to an initial search summary

An early web-search summary claimed "Claude Code v2.1.59+ features six memory subsystems with a
4-type taxonomy (user/feedback/project/reference)" — this specific framing doesn't appear anywhere
in the official docs read directly (`code.claude.com/docs/en/memory`,
`code.claude.com/docs/en/sub-agents`), which describe two complementary systems (`CLAUDE.md` +
auto memory) plus the subagent-scoped `memory` field's three scopes (`user`/`project`/`local` — not
"user/feedback/project/reference"). Treating the search-summary framing as unreliable/possibly
conflated with unrelated content; the docs-sourced description above supersedes it. Flagging this
explicitly since a prior run's design doc corrections note (see
`specs/2026-08-06-continuous-self-improvement-design.md`) established the pattern of noting when
something needs correcting rather than silently absorbing it.

## Candidate improvements (not yet acted on)

| Candidate | Impact | Effort | Notes |
|---|---|---|---|
| Install `chrome-devtools-mcp` | M | L | Recommended this run — see decisions/. Closes a real gap (live browser debugging, distinct from Playwright's test-automation role). |
| Install `typescript-lsp` | M | L | Recommended this run — matches Compass's actual TS stack, first-party. |
| Adopt `.claude/rules/` path-scoping for `~/.claude/CLAUDE.md` when it grows | M | L | Recommended this run, not urgent — current file is still small (3 short sections per baseline). Worth having documented before it's needed under time pressure. |
| Give a custom subagent `memory: project` scope | L (today) | L | Not actionable yet — no project defines custom subagents. Revisit when one does; the mechanism itself needs no further research, it's ready to use the moment there's a subagent to attach it to. |
| Apply the still-open effort-routing recommendation to `~/.claude/CLAUDE.md` | M | L | Not new this week — flagging that `decisions/2026-08-08-token-efficiency-effort-routing.md` is still pending manual review, two weeks out now. |
