# Weekly research — 2026-10-17

**Deep-dive topic:** MCP servers & integrations (rotation wraps back to 1st topic, after MCP
servers & integrations → agent architectures & multi-agent patterns → prompting techniques →
memory systems → testing/verification approaches → tooling/infrastructure/automation — one full
lap completed, this is lap 3's first topic).

## Skills scan

Re-added all six known marketplaces (ephemeral sandbox, per standing note in `baseline.md`):
`claude-plugins-official` (unchanged at 315), `superpowers-dev` (unchanged, 1 plugin),
`claude-code-workflows` (unchanged at 94), `impeccable` (unchanged, 1 plugin), `thedotmack`
(unchanged, 2 plugins), `claude-community` (**2284 → 2283, first observed shrink** — every prior
pass since this marketplace was added 2026-09-12 showed it growing or staying flat; one plugin
dropped out, not identifiable by name against a 2,283-entry listing with no per-entry timestamp
field, same "not acted on, doesn't track third-party removals" precedent as `claude-code-workflows`'s
95 → 92 drop on 2026-08-22).

Ran a full name-diff of `claude-plugins-official` against every name previously recorded in
`skills-seen.md` anyway (not just trusting the unchanged net count) — surfaced ~200 names still not
individually itemized by name in that file. This is **not new growth**: the net count has been
flat at 315 since 2026-10-03, and the 2026-10-03 entry already explicitly flagged this same
long-standing unitemized tail and decided not to re-itemize it line-by-line every week going
forward. Spot-checked a sample of the diff output and found nothing outside the already-established
buckets (credentialed third-party SaaS, cloud-provider suites, unused-language tooling) — consistent
with four consecutive flat/near-flat weeks for this marketplace. Not re-logging the same tail again
this week; see `skills-seen.md`'s 2026-10-03 entry for the full accounting.

Ran a narrow keyword sweep (`mcp`, `integration`, `connector`, `registry`) against `claude-community`
given this week's deep-dive topic — 498 of 2,283 entries matched, far too many to review
individually in a low-effort weekly pass (same conclusion as the 2026-10-10 tooling sweep's 429
matches). No individual plugin surfaced as independently reputable enough to flag; this marketplace
remains reputable only as a *host* (automated security scan, not a quality/maintainer bar), per the
standing 2026-09-12 caveat.

Web search for new installable marketplaces found nothing beyond the discovery-directory sites
already on record (`claudemarketplaces.com`, `aitmpl.com`) and generic how-to content repeating
`/plugin marketplace add owner/repo` syntax already known.

**Net result: nothing cleared the recommend bar this week.** No new plugin installed, recommended,
or individually rejected — the only genuinely new signal was the `claude-community` shrink noted
above, which isn't actionable.

## Token-efficiency / context-management check

Checked for anything new beyond what `baseline.md` already documents (prompt caching, harness-level
auto-compaction, Haiku model-routing + its pricing-cliff boundary, `Workflow` effort-level routing,
the `fork` subagent mode, `defer_loading`/Tool Search). Nothing cleared the quality bar this week:

- Confirmed (again) deferred/lazy tool-schema loading is working as documented — this session's own
  tool list arrived with `ListMcpResourcesTool`, `WebSearch`, `SendMessage`, and the GitHub MCP tools
  as deferred stubs, same mechanism found 2026-08-08 and reconfirmed 2026-10-03.
- General web coverage of the Agent SDK's context-management layers (server-side compaction, context
  editing / tool-result clearing at a 100K-token default threshold, the file-based memory tool) came
  up again — same platform/API-level mechanism already explicitly carved out of this check's scope
  per the design spec ("this check is about workflow-level technique... not re-litigating the
  platform's own internals"). One open GitHub feature request (per-agent compaction thresholds for
  `Agent`-tool subagents, since today's auto-compaction only triggers near ~95% of the model's
  window) is a real gap but nothing shipped yet — nothing to recommend.
- No new delegation/routing/compression pattern found. Checked the Claude Code v2.1.287–294 release
  notes again (already fully absorbed into `baseline.md` by the 2026-10-10 pass) for anything dated
  after Oct 8 — nothing newer turned up.

**No new workflow-level technique found this week that clears the quality bar.** Confirming pass —
no `decisions/` entry written.

## Deep-dive: MCP servers & integrations

Three findings, one of them a genuine baseline-accuracy correction:

1. **Native account-level connector/skill/plugin catalogs are richer — and more precisely
   queryable — than previously documented.** This session has direct access to `ListConnectors`,
   `SuggestConnectors`, `ListPlugins`, `SearchPlugins`, `ListSkills`, `SearchSkills`, and
   `SuggestSkills` — none of these individual tool names were previously recorded in `baseline.md`
   or any `research/` entry (the 2026-09-05 pass found `SearchMcpRegistry`, a sibling tool for
   searching the *uninstalled* connector directory, but not this fuller set covering what's already
   installed/enabled). Calling them this run surfaced a genuinely new structural distinction: these
   operate against **claude.ai's own account-level catalogs**, not Claude Code's CLI plugin-
   marketplace system (`claude plugin marketplace`/`install`, the thing `skills-seen.md` has been
   tracking all along) and not quite the same thing as the session's blended available-skills
   listing either. Specifically:
   - `ListPlugins` (claude.ai's own plugin catalog, distinct from CLI marketplaces) returned **zero
     results** for this account — this catalog is simply unused, a different mechanism from the CLI
     plugins already documented (`superpowers`, `playwright`, etc.), which it has no visibility into
     and vice versa.
   - `ListSkills` (claude.ai's personal Skills library) returned 14 entries — `unslop`, `docs`,
     `google-workspace`, `computer-use`, `chrome-browser`, `built-in-browser`, `deep-research`,
     `import-memory`, `morning`, `skill-creator`, `xlsx`, `pptx`, `pdf`, `docx`. This is a **strict
     subset** of this session's full available-skills listing — it excludes every Claude-Code-
     specific bundled skill (`dataviz`, `artifact-design`, `update-config`, `keybindings-help`,
     `code-review`, `simplify`, `fewer-permission-prompts`, `loop`, `claude-api`,
     `workflow-authoring`, `run`, `plugin-authoring`, `init`, `security-review`,
     `session-start-hook`), confirming those are supplied by the Claude Code harness itself, not the
     claude.ai personal-skills catalog `ListSkills` queries. Net: there are now three distinct
     skill/plugin surfaces, not two — (a) Claude Code CLI plugin marketplaces (self-managed, what
     this routine scans weekly), (b) claude.ai's account-level plugin catalog (unused, separate
     install path), (c) claude.ai's personal Skills library (overlaps partially with Claude Code's
     bundled-skill listing, but is a narrower, separately-queryable set). Worth recording since a
     future pass reasoning about "what skills does this account have" needs to know which of the
     three it's asking about.
2. **Baseline connector list was stale — corrected this run.** `baseline.md` documented "Currently
   connected: Supabase, Wallet (BudgetBakers), Gmail, Google Calendar, Google Drive." Calling
   `ListConnectors` directly (ground truth, not inference) this run shows: `Supabase` connected,
   `Wallet` connected, `Gmail` connected, **`Google Calendar` → `not_connected`**, **`Google Drive` →
   `not_connected`** (both contradict the baseline's "connected" claim), plus **two connectors not
   previously documented at all**: `Compassfi` (connected — presumably Compass's own project
   connector) and `Scalable Capital` (`connect_incomplete` — auth started but not finished). All
   seven show `enabledInChat: false` in this particular session (expected — this routine's sessions
   attach no MCP connectors by design, per the spec), which only means none of their tools are
   loaded *here*, not that they're disconnected at the account level. Updated `baseline.md`'s MCP
   section to reflect the corrected list.
3. **MCP protocol spec detail, no new revision found.** Re-checked for anything newer than the
   2026-07-28 revision `baseline.md` already documents (elicitation replacing `sampling`/`roots`/
   SSE logging, MCP Apps). Search results surfaced more granular detail on the same-era spec
   (OIDC Discovery, icons metadata, incremental scope consent, URL-mode elicitation specifically,
   sampling-as-tool-calling, OAuth Client ID metadata documents, experimental Tasks support) but
   nothing dated later or substantively new beyond what's already recorded as a maintenance-
   freshness signal. No baseline change needed here.

## Candidate improvements

| Candidate | Impact | Effort | Status |
|---|---|---|---|
| Correct `baseline.md`'s MCP connector list (`Compassfi`, `Scalable Capital` added; `Google Calendar`/`Google Drive` corrected to not-connected) | M | L | Done this run |
| Document the three distinct skill/plugin catalogs (CLI marketplaces / claude.ai plugin catalog / claude.ai personal Skills library) in `baseline.md` | L | L | Done this run |
| Revisit whether `SuggestConnectors`/`ListConnectors` should be used proactively during ordinary (non-routine) project work to surface real, already-connected integrations instead of guessing | L | M | Not acted on — no concrete driver yet, noted for a future pass |
| Investigate the claude.ai account-level plugin catalog (`SearchPlugins`/`SuggestPluginInstall`) as a possible alternative or supplement to CLI marketplace scanning for this routine's own weekly skills scan | L | M | Not acted on — currently empty/unused, no evidence yet that it would surface anything the CLI-marketplace scan doesn't; revisit if it becomes populated |
| Per-agent compaction threshold for `Agent`-tool subagents (today's auto-compaction triggers only near ~95% of window) | M | — | Not our lever — open upstream feature request, nothing to do here yet |

No `decisions/` entries this week — nothing cleared the recommend bar for either the skills scan or
the token-efficiency check. The two `baseline.md` corrections above are factual (ground-truth tool
output), not policy recommendations, so applied directly rather than routed through manual review,
same precedent as prior weeks' pure-fact baseline updates (e.g. the 2026-09-05 `SearchMcpRegistry`
addition).
