# Weekly research — 2026-08-29

**Deep-dive topic:** Tooling/infrastructure/automation (6th and last in rotation, after MCP servers
& integrations → agent architectures & multi-agent patterns → prompting techniques → memory
systems → testing/verification approaches). Next run wraps back to MCP servers & integrations.

## Skills scan

Ephemeral sandbox (per `baseline.md`) — re-added all five previously-configured marketplaces:
`claude-plugins-official`, `superpowers-dev` (obra/superpowers), `claude-code-workflows`
(wshobson/agents), `impeccable` (pbakaus/impeccable), `thedotmack` (thedotmack/claude-mem).

**Marketplace-level changes:**
- `claude-plugins-official`: **291 plugins, up from 286** (2026-08-22 count). Diffed the full
  current listing against every plugin name ever mentioned in `skills-seen.md`; the ~5 net-new
  entries weren't individually identifiable against last week's exact snapshot (this file only
  records counts, not full listings, week to week), but scanning the diff output found nothing
  that both (a) requires no account/credential and (b) maps to a workflow actually in use here —
  consistent with the pattern established since the 2026-08-08 bulk-rejection call (the long tail
  is overwhelmingly third-party SaaS integrations). Not itemized individually this week; revisit
  if a specific new name is independently cited as worth a look.
- `claude-code-workflows` (wshobson/agents): **93 plugins, up from 92** last week. Diffed against
  `skills-seen.md` and found 51 previously-unitemized names (the marketplace has grown enough
  since 2026-08-06/08 that a large chunk of its original ~95 was never individually named, only
  bulk-rejected by category). Individually evaluated below, prioritizing ones with an
  infrastructure/automation/security theme matching this week's deep-dive.
- `superpowers-dev`, `impeccable`: unchanged (1 plugin each).
- `thedotmack`: unchanged (2 plugins — `claude-mem`, `claude-mem-cowork`; the latter not
  previously itemized by name, but it's the same rejected `claude-mem` project's Cowork-specific
  variant, same reasoning applies, not logged as a separate row).
- Web search for new marketplace sources found **`tonsofskills.com`** (backed by
  `jeremylongshore/tons-of-skills-marketplace`, `ccpi` CLI installer, 471 plugins / 3,069 skills /
  347 agents) — this is the same maintainer's project already evaluated and rejected on 2026-08-08
  as `jeremylongshore/claude-code-plugins-plus-skills` (then 2.6k★, 471 plugins/3,069
  skills/347 agents at a smaller count), now rebranded with its own domain and CLI tooling but the
  same single-maintainer kitchen-sink shape and scale. The original rejection reasoning
  (per-plugin vetting infeasible at this scale for a low-effort weekly routine, no independent
  quality signal beyond the maintainer's own validation claims) still applies — not re-added, not
  logged as a new marketplace (same source, reaffirmed rather than newly found).

**New plugins evaluated this run** (from `claude-code-workflows`'s 51 previously-unitemized names,
filtered toward this week's tooling/infrastructure/automation theme; the remainder bulk-noted):

- **`hol-guard`** (Hashgraph Online, v0.1.0) — the single most interesting find this week,
  conceptually. "Local pre-execution security controls and pre-install scanning for AI agent
  tools, packages, skills, plugins, and MCP servers" — i.e., automated vetting of exactly the kind
  of thing this repo's own weekly skills scan does by hand. Read into what it actually does: it
  installs a `PreToolUse`-style enforcement hook that routes tool calls through an async local (or
  optionally cloud/OAuth) Guard review and **fails closed** — a timeout, missing Guard command, or
  rejected approval blocks the tool call outright. No API key required for local mode. **Rejected**
  despite the appealing concept: v0.1.0, 4 stars, 1 fork, 73 commits — this is a materially larger
  automatic-interception footprint (it gates *every* tool call, not just session start or specific
  file edits) with *less* track record than any of the memory tools already rejected on this exact
  concern (`pensyve`, `remember`, `claude-mem` — all of which only run background hooks, none of
  which block tool execution outright). Flagged as a genuine revisit candidate if it matures past
  early-alpha — the mission overlap with this repo is real — but "matches our mission" isn't a
  substitute for "has a track record," same bar applied to everything else in this category.
- `cicd-automation`, `kubernetes-operations`, `cloud-infrastructure`, `deployment-strategies`,
  `observability-monitoring`, `application-performance`, `database-cloud-optimization`,
  `incident-response`, `framework-migration`, `database-design`, `security-scanning`,
  `security-compliance`, `backend-api-security`, `frontend-mobile-security` — **rejected (bulk)**,
  no mapped current-project workflow: no project tracked here runs a CI/CD pipeline, Kubernetes
  cluster, cloud infrastructure project, incident-response process, database-design/migration
  effort, or compliance program to attach any of these to. Same "no clearly mapped workflow" bar
  used for every prior wshobson domain-specific rejection.
- `code-refactoring` — rejected, overlaps bundled `simplify` skill.
- `error-debugging` — rejected, overlaps superpowers' `systematic-debugging` (+ already-rejected
  `debugging-toolkit`/`distributed-debugging`/`error-diagnostics`).
- `documentation-standards`, `code-documentation` — rejected, no active docs-generation project.
- `llm-finetuning`, `machine-learning-ops`, `data-engineering` — rejected, no ML/data-infra
  project.
- `python-development`, `systems-programming`, `jvm-languages`, `web-scripting`,
  `functional-programming`, `julia-development`, `arm-cortex-microcontrollers`,
  `shell-scripting`, `dgx-spark-ops`, `dotnet-contribution` — rejected (bulk), language/hardware
  bundles for stacks not in use anywhere, same precedent as the 2026-08-15 LSP-family bulk
  rejection.
- `seo-content-creation`, `seo-technical-optimization`, `seo-analysis-monitoring`,
  `business-analytics`, `startup-business-analyst`, `hr-legal-compliance`,
  `customer-sales-automation`, `content-marketing`, `social-publishing`, `hermes-tweet`,
  `blockchain-web3`, `quantitative-trading`, `payment-processing`, `game-development` — rejected
  (bulk), no mapped workflow, same as the 2026-08-08 general grab-bag bulk rejection.
- `accessibility-compliance` — rejected, overlaps already-installed `frontend-design`/
  `modern-web-guidance`'s accessibility-floor coverage.
- `reverse-engineering` — rejected, no mapped current project (no malware-analysis/CTF need
  identified anywhere in this setup).
- `file-conversion` — rejected. "Convert files between 999 routes... via the free ChangeThisFile
  API... no API key required" — no credential needed, but it routes file *content* through an
  unvetted third-party API by design for every conversion; that's a real data-handling concern
  independent of the credential question, and no unvetted-third-party-network-call precedent has
  been accepted here. No mapped workflow need either.
- `c4-architecture`, `multi-platform-apps` — rejected, no mapped current-project workflow.

**Net result: nothing cleared the recommend bar this week.** See `skills-seen.md` for the full
dated log.

## Token-efficiency / context-management check

Researched current 2026 material: Anthropic's context-editing + memory-tool combination (already
noted in a prior pass as a platform-level, not-actionable-here mechanism); general "reduce token
usage" roundups converging on the same handful of habits already reflected in
`~/.claude/CLAUDE.md`'s token-efficiency section (targeted reads, model-tier routing) or already
drafted-and-pending (effort-level routing for `Workflow` dispatch, path-scoped `.claude/rules/`
files). One new-looking item surfaced: **"Context Mode"**-style plugins (e.g. the one referenced
in `hesreallyhim/awesome-claude-code` issue #829, and a separate independent project,
`egorfedorov/claude-context-optimizer`) that intercept MCP tool output, index it locally, and hand
Claude a compact summary instead of raw JSON — claimed 50–90%+ token reduction specifically in
MCP-heavy sessions. **Not recommended**: (a) unclear provenance/maturity on a quick check (small
individual-maintainer repos, promotional-sounding claims, no independent verification found), (b)
doesn't map to a real current need — this account's MCP usage is the platform connector system
(Supabase/Wallet/Gmail/Calendar/Drive) rather than a local `.mcp.json` server generating heavy raw
output in a Claude Code session, so there's no session here that would actually exercise the
claimed savings. Logged as a watch item rather than a recommendation.

**No new technique found this week that clears the quality bar and isn't already
recommended-and-pending.** Worth naming explicitly: `context7` has now been sitting as
"recommended, not yet installed" since 2026-08-08 (3 weeks), `chrome-devtools-mcp` and
`typescript-lsp` since 2026-08-15 (2 weeks), and the effort-routing/path-scoped-rules
token-efficiency drafts from the same two dates are also still unapplied. None of this is a
problem with the research routine — the manual-review half of the system just hasn't had a pass
yet — but it's a growing backlog worth flagging rather than silently piling higher.

## Deep-dive: Tooling/infrastructure/automation

Compared against what `baseline.md` currently documents (hooks: superpowers' one `SessionStart`
hook, the `block-no-verify` `PreToolUse` hook; sandbox bootstrap pain: fresh cloud sessions
reinstalling Deno/Supabase CLI/Playwright/`gh` from scratch every time, called out explicitly as
unresolved in the "Sandbox bootstrap notes" section).

**Finding 1 — a bundled skill exists that directly addresses the documented sandbox-bootstrap
pain, and it wasn't on `baseline.md`'s radar.** This session's own available-skills listing
includes `session-start-hook`: "Creating and developing startup hooks for Claude Code on the web.
Use when the user wants to set up a repository for Claude Code on the web, create a SessionStart
hook to ensure their project can run tests and linters during web sessions." This is close to
exactly the gap `baseline.md`'s own "Sandbox bootstrap notes" section names — Compass and this
repo both currently re-derive their tooling setup from scratch on every fresh cloud checkout, with
no committed automation for it (Compass's fix so far is a written doc, `docs/environments.md`,
describing the steps by hand, not a hook that runs them). No install needed — it's platform-bundled.
**Actionable**: worth using on Compass's next cloud/web session to actually convert
`docs/environments.md`'s manual steps into a real `SessionStart` hook, and worth considering for
this repo's own (much smaller) bootstrap needs (marketplace re-adds, `gh`/git auth) if this
routine's setup steps ever grow past what's reasonable to repeat by hand each firing.

**Finding 2 — the broader bundled-skill roster has drifted from what `baseline.md` documents, in
more than one place.** Comparing this session's actual available-skills listing against
`baseline.md`'s "Bundled (platform-provided, not self-managed)" list found skills present now that
aren't listed there: `session-start-hook` (above), `design` (canvas/artboard design tool, distinct
from the `design-sync`/`design-login` DesignSync-backed skills already documented),
`workflow-authoring` (the reference skill backing the `Workflow` tool documented under Agent
architecture — its existence as a *named skill* wasn't previously called out), and
`docx`/`import-memory`/`morning`/`pdf`/`pptx`/`skill-creator`/`unslop`/`xlsx`. Conversely, some
skills `baseline.md` lists (`schedule`, `claude-in-chrome`, `design-sync`, `design-login`, and
`review` — this session shows `code-review` instead, not `review`) aren't visible in this
session's listing. The likely explanation is that skill visibility is itself environment/session
dependent (connector- or feature-gated capabilities not present in this particular ephemeral cloud
sandbox), not that anything was actually removed — but this wasn't verified, and `review` vs.
`code-review` in particular could be a genuine rename worth confirming, since a "duplicate of
`review`" rejection reasoning appears more than once in `skills-seen.md` and would need updating
to reference the current name if so. Flagged in `baseline.md` below as a note to reconfirm during
a normal (non-scheduled, persistent-environment) session rather than asserted as fact from this
sandbox alone.

**Finding 3 — 2026 hook-based infrastructure-automation patterns (Terraform/Kubernetes/cloud-CLI
wrapping via `PreToolUse`/`PostToolUse`, `Stop`-hook-triggered PR creation) are real and
documented, but nothing here has a project to attach them to.** No project tracked in this repo
runs Terraform, Kubernetes, or a cloud-infra pipeline that would benefit from hook-wrapped
`terraform validate`/`kubectl apply` guardrails. PR-creation-via-`Stop`-hook is also redundant with
what's already true here: this harness's own built-in commit/PR workflow (spelled out directly in
the system prompt, and exercised normally by this session's own git operations) already covers
that without a hook. This matches general best-practice guidance found this week too ("avoid hooks
that duplicate tooling... add hooks for stable, repeated, high-value automation," "hooks run with
full user permissions — CI, branch protection, and infra controls should remain the authoritative
enforcement layers, not hooks"). **Confirming, not gap-closing**: current reliance on
`block-no-verify`'s single narrow hook plus the harness's native git/PR workflow is already the
right shape for what's actually in use; nothing found this week argues for adding more hooks
speculatively.

**Overall read:** this rotation's most concrete finding wasn't from web research at all — it was
noticing that `baseline.md`'s own picture of *this session's available tooling* had gone stale,
inside a repo whose entire purpose is keeping that picture current. The `session-start-hook`
skill in particular is a genuine, no-install, directly-actionable fix for a pain point this repo
has been carrying as an open item since 2026-08-06.

## Candidate improvements (not yet acted on)

| Candidate | Impact | Effort | Notes |
|---|---|---|---|
| Use `session-start-hook` on Compass's next cloud/web session to convert `docs/environments.md`'s manual bootstrap steps into a real `SessionStart` hook | M | L | No install needed, bundled skill. Directly closes a pain point named in `baseline.md`'s "Sandbox bootstrap notes" since 2026-08-06. |
| Reconfirm the bundled-skill roster (`review` vs. `code-review`, and whether `schedule`/`claude-in-chrome`/`design-sync`/`design-login` are genuinely absent vs. just not surfaced in this ephemeral sandbox) from a normal persistent-environment session, and correct `baseline.md`/`skills-seen.md` references if `review` really was renamed | L | L | This run's finding is sandbox-sourced and not independently confirmed; worth a quick check rather than acting on it further from here. |
| Act on the aging manual-review backlog: `context7` (recommended 2026-08-08, 3 weeks pending), `chrome-devtools-mcp`/`typescript-lsp` (2026-08-15, 2 weeks pending), and the effort-routing/path-scoped-rules `~/.claude/CLAUDE.md` drafts (same two dates) | M | L | Not a new finding — a process nudge. All the reasoning is already written; this is purely "someone needs to actually run the manual review pass." |
| Revisit `hol-guard` once it has real adoption/track record — its mission (pre-install vetting of AI tooling) directly mirrors this repo's own | L (speculative) | L | v0.1.0, 4★/1 fork today; too early regardless of concept fit. |
