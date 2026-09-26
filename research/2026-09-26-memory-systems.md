# Weekly research — 2026-09-26

**Deep-dive topic:** Memory systems (2nd time through the rotation — MCP servers & integrations
(2026-09-05) → agent architectures & multi-agent patterns (2026-09-12) → prompting techniques
(2026-09-19) → **memory systems** (this run, first covered 2026-08-15)).

## Skills scan

Re-added all six previously-known marketplaces this run (`claude-plugins-official`,
`superpowers-dev`, `claude-code-workflows`, `impeccable`, `thedotmack`, `claude-community`) — the
routine's cloud sandbox is ephemeral, so nothing persisted from 2026-09-19; see `baseline.md`'s
"Weekly research routine" section.

- `superpowers-dev`, `impeccable` — unchanged (1 plugin each).
- `thedotmack` — unchanged (2 plugins, both still rejected).
- `claude-code-workflows` (wshobson/agents) — unchanged at 94 plugins, all already
  itemized/bulk-rejected.
- `claude-community` — unchanged at 2,282 plugins. Not re-scanned in full this run either
  (same "narrow, deep-dive-driven searches, not a full re-scan" approach established 2026-09-12);
  no keyword search against it turned up anything memory-systems-relevant worth a plugin row.
- `claude-plugins-official` — grew 310 → 314. Five genuinely new names (not appearing anywhere in
  this file's prior bulk-row prose either) individually evaluated: `duckdb-skills` (local
  analytics DB, no credential, but no mapped data-engineering workflow), `gitkraken` (redundant
  with git used directly), `expo` (React Native — no tracked project uses it), `convex`
  (backend-as-a-service, requires an account), `mlflow` (ML experiment tracking, no ML pipeline
  tracked here). All rejected — see `skills-seen.md` for the one-line reasoning on each. The
  remaining growth spot-checked into the same established SaaS/cloud-provider/per-language-tooling
  buckets, nothing novel.

Web search for other reputable marketplaces not yet configured found one genuine new candidate:
**`netresearch/claude-code-marketplace`** (github.com/netresearch/claude-code-marketplace) — a
company-backed (Netresearch DTT GmbH), actively maintained collection (61★/14 forks/366 commits,
GitHub Actions CI, Renovate, structured `docs/decisions/`) of 39 skills. Evaluated and **not
added**: every skill targets TYPO3, PHP, Go, Docker, or Jira — no tracked project uses any of that
stack (Compass is TypeScript/Next.js; this repo is markdown-only) — so adding it would mean
permanent future-scan overhead for zero actionable payoff. Worth remembering for a different
reason, though: its skills are built on the **`agentskills.io` open standard**, explicitly designed
to be portable across Claude Code, Cursor, Copilot, Codex, and Gemini CLI via `AGENTS.md` rather
than a Claude-specific format. That connects directly to this week's deep-dive finding below.
Also found `dashed/claude-marketplace`, explicitly self-described as "a local marketplace for
personal Claude Code skills" — same no-reputability-signal-beyond-a-search-hit bar that sank
`xiaolai/claude-plugin-marketplace` (2026-08-06), not added.

## Token-efficiency / context-management check

No technique cleared the quality bar for a `~/.claude/CLAUDE.md` recommendation this run. Two
things worth recording as "checked, not applicable" rather than silently skipped:

- **Anthropic's Context Editing API + Memory Tool** (Messages API, beta since mid/late-2025,
  reconfirmed still current) — server-side clearing of stale tool-use/result pairs and a
  file-based memory tool for long-running agents built directly on the API, with Anthropic-reported
  gains (~29% from context editing alone, ~39% combined with the memory tool) over an unmanaged
  baseline. This is squarely a **Messages-API-level** feature for people building their own agent
  on top of the raw API — it's what `claude-api`/`agent-sdk-dev`-type work would use, not something
  a Claude Code CLI session configures directly (Claude Code already does its own automatic prompt
  caching and auto-compaction, which is the out-of-scope "platform-managed internals" this check's
  own carve-out excludes from a weekly re-litigation). No tracked project builds a standalone
  Agent-SDK/Messages-API app (confirmed already — see the 2026-09-12 `agent-sdk-dev` rejection) so
  there's nothing to apply this to right now regardless.
- **"Dreaming" / memory consolidation** — this term keeps surfacing in search results in a way that
  reads, at a glance, like it applies to Claude Code's own auto memory. It does not: it's a
  research-preview feature of the separate **Managed Agents API** product (consolidates an agent's
  memory store between sessions — merging duplicates, dropping stale entries). The 2026-08-15
  memory-systems pass already made this exact distinction once; this run reconfirms it still
  holds and flags the conflation risk again for future passes, since it's evidently an easy mistake
  for search-summary content to make repeatedly.

## Deep-dive: memory systems

Read Claude Code's official memory docs (`code.claude.com/docs/en/memory`) directly end-to-end
again, rather than trusting search-summary content — the same discipline the 2026-08-15 pass
adopted after finding imprecise claims in a first search pass (and this run's own search results
repeated exactly that pattern with the "Dreaming" conflation above).

**Confirmed unchanged from the 2026-08-15 baseline:** auto memory's storage location, structure
(`MEMORY.md` index + on-demand topic files), 200-line/25KB load cap, machine-local scope, and the
subagent-level `memory: user|project|local` field (still not actionable — no project here defines
custom subagents).

**New since the 2026-08-15 pass, not yet reflected in `baseline.md` before this run:**

1. **Native `AGENTS.md` support (Claude Code v2.1.277+).** Claude Code can now read a repository's
   `AGENTS.md` directly as project instructions — no `CLAUDE.md` import or symlink workaround
   needed — when there's no `CLAUDE.md`/`CLAUDE.local.md` on the path. A `/config` → "Project
   instructions" setting controls the precedence (`AGENTS.md` only, `CLAUDE.md` only, both
   together, or managed-only). A companion `/import` command (v2.1.213+) one-time-copies another
   coding tool's config (Cursor rules, `.github/copilot-instructions.md`, `AGENTS.md`, etc.) into
   the matching `CLAUDE.md`, carrying over MCP servers/commands/subagents/skills too. This is
   genuinely new mechanism, not a minor refinement — added to `baseline.md`'s Global config
   section. **Not currently actionable**: no tracked project uses `AGENTS.md` or another AI coding
   tool alongside Claude Code. Worth revisiting if that changes, and it's the same portability
   angle behind this week's `netresearch/claude-code-marketplace` finding above — if a future
   project wants skills/instructions portable across multiple AI coding tools, `AGENTS.md` +
   `agentskills.io`-style skills is the concrete mechanism for that, not something to build custom.
2. **`MEMORY.md` over-limit handling is now an explicit hard error, not silent truncation.** When
   Claude writes to `MEMORY.md` and the result exceeds the 200-line/25KB cap, the write still
   succeeds but Claude Code returns an error telling Claude to rewrite the index (move detail into
   topic files, merge/drop stale entries) — previously everything past the limit was just silently
   dropped on the next load. A safety improvement, not something requiring any behavior change here
   since Claude already self-curates `MEMORY.md` per the documented mechanism.
3. **`permissions.blockReadsOutsideWorkingDirectories` now also gates `autoMemoryDirectory`.** When
   that permission is on, Claude Code loads no auto memory from — and saves none to — a
   repo-chosen memory directory that resolves outside the working directory, regardless of where it
   points. Not applicable here: no project sets a custom `autoMemoryDirectory` in a repo-supplied
   settings file.

Nothing else found rises above "minor documentation clarification" (the `.claude/rules/` symlink/
external-import approval nuances, `CLAUDE_CODE_PROJECT_DIR_NAME` for sharing one memory directory
across differently-named checkouts, managed-policy `CLAUDE.md` for org-wide deployment) — read, none
actionable for this repo's tracked projects.

## Candidate improvements (not already acted on)

| Candidate | Impact | Effort | Notes |
|---|---|---|---|
| Adopt `AGENTS.md` (possibly alongside `CLAUDE.md`) if a tracked project starts using another AI coding tool | L | L | No current driver — pure awareness item until that happens. |
| Revisit `netresearch/claude-code-marketplace` if a tracked project adopts TYPO3/PHP/Go/Docker/Jira | L | L | Purely stack-contingent; no action now. |
| Revisit `duckdb-skills` if a tracked project starts a data-engineering/analytics workflow | L | L | No credential blocker, just no current use case. |
| Apply the still-open 2026-08-08 effort-routing and 2026-09-05 fork-subagent token-efficiency recommendations to `~/.claude/CLAUDE.md` | M | L | Not new this run — carried forward; both are still pending manual review, flagging again since they've now been open for 7 and 3 weeks respectively. |
