# Ad hoc research — 2026-08-06 (manual pass, not the scheduled routine)

**Trigger:** user explicitly asked to broaden scope beyond skills alone (state-of-the-art
AI-assisted dev/autonomous coding agents generally) and to add token efficiency as an ongoing,
dedicated concern — not just one line item buried in a rotation topic.

## Token efficiency & context management — state of the art

- **Prompt caching**: Anthropic's automatic caching moves the cache point forward as a
  conversation grows; stable content early (system prompt, tool defs) maximizes hit rate. Reported
  41–90% cost reduction on cached tokens depending on setup. Largely platform-managed in this
  harness already — this session's own system prompt explicitly documents the 1-hour cache TTL and
  explicitly warns against scheduling extra wakeups just to "keep the cache warm," since that's
  automatic. **No action needed here — already correctly aligned.**
- **Context editing / pruning**: clearing stale tool results and thinking blocks rather than
  summarizing, when old output is no longer relevant — the harness already auto-compacts as context
  fills (per this session's own system prompt). **Already handled at the platform level.**
- **Scoped exploration via subagents/forks**: the standard pattern is keeping exploratory/read-heavy
  work out of the main context entirely via subagent dispatch, so the parent doesn't pay for
  re-reading it later. This session already does this consistently (forking for research, `Explore`
  agent for codebase search). **Already good practice — confirmed, not a gap.**
- **Model routing for subagents**: the clearest actionable gap found. Common pattern: route cheap,
  high-volume, low-reasoning subtasks (classification, simple lookups, routing) to a cheaper/faster
  model, reserve the frontier model for the main loop and genuinely hard reasoning. The `Agent` tool
  available in this harness has a `model` parameter (`sonnet`/`opus`/`haiku`/`fable`) that
  **defaults to inheriting the parent's model when omitted** — in practice, every subagent
  dispatched this session inherited the parent model rather than being explicitly routed to
  something cheaper for simple work. **Real, concrete, low-effort improvement — see decision.**

## Broader agentic-coding landscape (2026)

- The field has shifted from short chat-style prompt/response to long autonomous execution loops
  (agents running minutes-to-hours, multi-file edits, opening PRs, running tests with limited
  supervision) — this session's own work (the Compass sync feature, the dev-toolkit build-out, the
  scheduled routine) already fits this pattern, not a laggard here.
- Competitive landscape as of late July 2026: Claude Code (Opus 5) ranked #1, Codex (GPT-5.6 Sol)
  #2, with Cursor, Gemini CLI, Grok Build, Kimi K3, Copilot, Devin, Windsurf also cited as
  significant. Per-subagent model control (routing different subagents to different models) is
  specifically called out as a Claude Code strength — directly reinforces the model-routing finding
  above: it's a real differentiator being underused here, not a hypothetical one.
- No indication of a fundamentally different agent architecture pattern (e.g. a new dominant
  planning paradigm) that this setup is missing — the gap found this pass is tactical (model
  routing, a place for standing instructions to live) rather than architectural.

## A structural gap this pass surfaced

Every decision this repo records is genuinely durable *as history* (git), but nothing automatically
re-reads `dev-toolkit` at the start of a session the way a project's own `CLAUDE.md` gets loaded.
A *behavioral* decision (like "route simple subagent work to Haiku") had no mechanism to actually
change future behavior — only plugin installs did (those persist through `~/.claude/plugins`
itself). Fixed by establishing `~/.claude/CLAUDE.md` (global, loads in every project's session,
previously nonexistent — noted as a gap in `baseline.md` since this repo's creation) as the target
for standing instructions distilled from `dev-toolkit` decisions. `dev-toolkit` remains the
reasoning/history; `~/.claude/CLAUDE.md` becomes the short, current-state summary that's actually
consulted. See `decisions/2026-08-06-global-claude-md.md`.

## Candidate improvements

| Candidate | Impact | Effort | Status |
|---|---|---|---|
| Route simple/high-volume subagent dispatches to `haiku` explicitly | M–H (cost, scales with subagent usage) | L | **Applied** — see `decisions/2026-08-06-global-claude-md.md` |
| Establish `~/.claude/CLAUDE.md` as the standing-instruction surface | H (unblocks every future behavioral decision, not just this one) | L | **Applied** |
| Re-check prompt-caching/context-editing guidance periodically as the harness's own auto-compaction logic evolves | L (already well-handled) | L | Deferred — revisit only if the harness's own behavior changes |
| Deeper comparative look at Devin/Cursor/Windsurf-style fully-autonomous PR workflows vs. this session's more supervised style | M | M | Deferred to a future rotation pass — no evidence current supervision level is wrong, just not deeply compared yet |
