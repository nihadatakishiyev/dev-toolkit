# Weekly research — 2026-09-19

**Deep-dive topic:** Prompting techniques (rotation's 3rd topic, revisited — MCP servers &
integrations → agent architectures → prompting techniques → memory systems → testing/verification
approaches → tooling/infrastructure/automation → MCP servers & integrations (2026-09-05) → agent
architectures (2026-09-12) → prompting techniques again, this run). First revisit of this specific
topic since its first pass on 2026-08-08.

## Skills scan

Re-added all six known marketplaces (ephemeral sandbox, per standing note in `baseline.md`):
`claude-plugins-official` (**grew 295 → 310**, +15), `superpowers-dev` (unchanged, 1 plugin),
`claude-code-workflows` (unchanged, 94 plugins — matches the count recorded since 2026-09-05),
`impeccable` (unchanged, 1 plugin), `thedotmack` (unchanged, 2 plugins), `claude-community`
(unchanged, 2,282 plugins — first time this marketplace's count hasn't grown between passes).

**`claude-plugins-official`'s unitemized tail.** Diffing the full current name list against every
plugin/marketplace name already recorded in `skills-seen.md` turned up 238 names never
individually evaluated before (not just the +15 net growth — a large previously-unitemized tail,
same situation the 2026-08-08/2026-08-29 bulk-reject entries already describe for this
marketplace). Read every name + description rather than assuming; the overwhelming majority split
into the same buckets already established:

- **Third-party SaaS/service integrations requiring an account or API key** (the large majority):
  Stripe, Shopify, Salesforce, HubSpot, Slack, Notion, Discord, Linear, GitLab, Sentry, Vercel,
  Netlify, PayPal, Twilio, Zapier, Postman, Mintlify, Langfuse, Tavily, Firecrawl, HuggingFace,
  Databricks, Snowflake, MongoDB, dozens of others across CRM/payments/observability/cloud-data/
  marketing/HR-legal/blockchain/gaming categories — fails the no-new-credentials criterion by
  construction, same as the 2026-08-08 bulk entry for this same marketplace.
- **Cloud-provider-specific suites** (AWS ×8, Azure ×3, GCP/BigQuery/Spanner/Firestore ×6+,
  Oracle ×3, SAP ×5) and **infra-as-code** (`terraform`) — no tracked project uses any of these
  platforms or IaC.
- **Language/framework-specific LSPs and dev kits** for languages not in use anywhere tracked here
  (`lua-lsp`, `swift-lsp`, `csharp-lsp`, `php-lsp`, `ruby-lsp`, `kotlin-lsp`, `jdtls-lsp`,
  `rust-analyzer-lsp`, `clangd-lsp`, `gopls-lsp`, `liquid-lsp`, `qt-development-skills`, etc.) —
  same precedent as the 2026-08-15 LSP bulk-rejection (only `typescript-lsp` recommended, Compass's
  actual stack).
- **Individually checked, not bulk:** `supabase` — redundant with the platform connector already
  connected for Compass (per baseline's MCP servers section); this plugin is a project-scoped MCP
  alternative requiring its own service-role/access-token configuration where the connector needs
  none, so strictly worse for the credential bar, not better. `browser-use` — already evaluated and
  rejected 2026-08-06 (competes with installed `playwright`); reconfirmed same reasoning, not
  re-itemized. `code-modernization` (legacy COBOL/Java/C++ migration workflow) — no legacy-migration
  project tracked. `playground` (interactive HTML-explorer generator) — doesn't map to a named
  workflow category (coding/research/planning/testing/deployment/data work), closer to a demo tool.
  `github` (official GitHub MCP server) — this session's own remote environment already gets
  repo-scoped `mcp__github__*` tools supplied directly, a distinct mechanism from this
  marketplace-installable plugin (see baseline's MCP servers section); no identified gap this would
  close for local/persistent-environment work that `gh` CLI doesn't already cover.

No plugin recommended from `claude-plugins-official` this run. Not itemized row-by-row in
`skills-seen.md` (238 names, same reasoning-bucket precedent already established) — logged as one
bulk entry.

**`claude-community` narrow keyword search** (per the 2026-09-12 pass's stated intent — narrow,
topic-driven searches against this 2,282-plugin marketplace instead of a full re-scan): searched
for prompting-technique-relevant terms (`prompt`, `meta-prompt`, `chain-of-thought`, `reasoning`,
`few-shot`, `step-back`). Far too many matches to hand-vet in a low-effort pass (40+ hits, mostly
generic "reasoning" substring matches on unrelated plugins — multi-agent frameworks, personality
role-play tools, workflow-pipeline skills). None independently stood out as both reputable
(visible maintainer/star signal — this marketplace's listing doesn't expose that metadata) and
mapped to an actual gap. No plugin recommended from this marketplace this run either.

**Marketplace search (web):** re-confirmed the same two marketplaces (`claude-plugins-official`,
`claude-community`) as Anthropic's official pair, and the same two discovery-directory sites
(`claudemarketplaces.com`, `aitmpl.com`, `buildwithclaude.com`) already on record — no genuinely
new *installable* marketplace found. Two curated GitHub "awesome-list" repos surfaced
(`ComposioHQ/awesome-claude-skills`, `travisvn/awesome-claude-skills`) — same "curated directory,
not an installable marketplace" category as `claudemarketplaces.com`, not added.

**Discovered but not from a configured marketplace — `AqueGen/model-routing`:** a third-party
Claude Code plugin implementing pinned model-tier subagents (Sonnet scout/implementer, Haiku
test-runner, Opus reviewer) plus a routing skill and a dispatch counter — directly on-topic for
this account's existing Haiku-routing practice. Checked reputability: **3 GitHub stars**, most
recent releases dated 2026-09-17 and 2026-09-19 (days-old, essentially solo-maintainer, still
actively iterating on its own bugs per open PRs). Same "too new/thin a track record" bar that sank
`superself` (2026-09-05, 4 stars/~2 weeks old). Not added as a marketplace, not recommended — noted
as a revisit candidate if it matures, consistent with how `superself`/`claude-mem`/`hol-guard` are
tracked.

## Token-efficiency / context-management check

Two findings, neither clears the "not yet reflected + actually changes workflow behavior" bar:

- **Claude Code's September 2026 update raised subagent/Task-tool output limits from a hardcoded
  32K-character default to 128K** (`bashOutputMaxChars`/`taskOutputMaxChars`), fixing a real
  failure mode where `CLAUDE_CODE_MAX_OUTPUT_TOKENS` wasn't honored by subagent calls and large
  subagent output (e.g. a big review or analysis) could silently truncate. This is a **platform
  capacity increase, applied automatically** — not a technique this account's own workflow needs to
  adopt or instruct differently, consistent with this routine's standing non-goal of not
  re-litigating harness-managed internals. Noted here as a fact worth knowing (a previously-invisible
  truncation risk in `Workflow`/subagent fan-out pipelines with large structured output is now less
  likely), not a recommendation.
- **The measured "~436k-token fixed subagent overhead" claim** (system-prompt + tool-definition cost
  a fresh subagent pays before reading anything) surfaced in a 2026 cost-analysis piece, cited as
  justification for routing trivial/bounded work directly rather than through a subagent. This is
  already this session's own default behavior (see this system prompt's own Explore-agent guidance:
  "if the target is already known, use the direct tool... reserve the Agent tool for open-ended
  questions") and already reflected in `~/.claude/CLAUDE.md`'s existing "prefer targeted reads over
  whole-file reads" rule per `baseline.md` — a confirming/quantifying data point, not a new gap.

No new technique recommended this week. No `decisions/` entry drafted for token efficiency.

## Deep-dive: Prompting techniques

Revisiting this topic six weeks after the 2026-08-08 pass, which found current practice already
substantially aligned with 2026 SOTA (spec-driven development, PLAN/ACT/VERIFY/ITERATE framing,
meta-prompting/step-back decomposition) and filed a confirming, not gap-closing, verdict. This
pass's findings are consistent with that verdict holding — no contradicting evidence found — plus
one genuinely new piece of vocabulary worth capturing:

1. **"Loop Engineering" — an Anthropic-published named framing** (`claude.com/blog/`, blocked by
   this session's egress proxy like other `claude.com` posts in prior passes; corroborated across
   multiple independent secondary sources, not single-sourced) that ties together primitives already
   separately documented in `baseline.md` — hooks, `CLAUDE.md`, skills, slash commands, the native
   `/goal` command, and scheduled routines (`/loop`, `/schedule`) — under one umbrella concept:
   writing *persistent* instructions/skills/hooks that make an agent iterate autonomously (gather
   context → act → verify → correct) until a defined completion condition holds, rather than relying
   on better single-turn prompting alone. **This is, concretely, what this repo's own weekly research
   routine already is** — a scheduled, self-verifying, autonomous loop — but the connection to this
   specific named pattern/term wasn't previously drawn in `baseline.md`. No behavior change indicated
   (the routine already does this); worth citing the term itself for future searches/comparisons, and
   worth noting that `/goal` + `/loop` + hooks are the concrete building blocks Anthropic itself names
   for this pattern, reinforcing (not superseding) the existing "documented, underused" flag already
   on the `Workflow` tool's Generator-Verifier idiom in the Testing/verification section — loop
   engineering and generator-verifier are adjacent but distinct (iterate-until-done vs.
   produce-then-independently-check).
2. **"Context engineering" as the successor framing to "prompt engineering"** — widely discussed in
   2026 sources as the shift from optimizing *how you ask* to architecting *what the agent knows when
   it answers* (persistent context pipelines, not just turn-level wording). Already substantially
   reflected in this account's actual infrastructure: `CLAUDE.md` (global + per-project), auto memory,
   `.claude/rules/` path-scoped files, and this very repo's own design (a durable external memory the
   routine reads before acting) are all context-engineering mechanisms already in place and already
   documented in `baseline.md`'s Global config section. A confirming observation, not a new gap — the
   terminology is newer than the practice here.
3. **No changes found to core prompting technique guidance itself** (XML-tag structuring, role
   prompting, step-by-step reasoning requests, few-shot example calibration, explicit
   uncertainty-permission) — every source found in this pass restates the same guidance already
   current as of the 2026-08-08 pass. Nothing indicates Claude 5-family models need materially
   different prompting technique than what's already reflected in how this account's own sessions
   operate (spec-driven planning, verification-before-completion, targeted context).

**Net assessment: confirming pass, same character as 2026-08-08's own verdict for this topic** — no
prompting-technique gap found, no plugin/config change indicated. The one actionable output is
naming "loop engineering" in `baseline.md` so it's recognized rather than re-discovered next time
this topic comes up, and so its relationship to the existing-but-unexercised Generator-Verifier gap
is on record in one place.

## Candidate improvements

| Candidate | Impact | Effort | Status |
|---|---|---|---|
| Add "loop engineering" framing + its relationship to the existing `/goal`/`/loop`/Generator-Verifier notes to `baseline.md` | L | L | Done this run (see below) |
| Update `claude-plugins-official` marketplace count (295 → 310) and note the 238-name bulk-reject in `baseline.md` | L | L | Done this run |
| Actually exercise the `Workflow` tool's Generator-Verifier (adversarial-verify) idiom on a real task | M | M | Still open — fourth consecutive research pass (two testing-topic, one agent-architecture, this one) flagging the same "documented, unused" gap; genuinely due for a forced try next time a verification-shaped task comes up |
| Revisit `AqueGen/model-routing` if it grows a real track record (currently 3★, days-old) | L | L | Not actionable yet — noted for future scans |
| Revisit `claude-community` with narrow, topic-driven keyword searches (continued from 2026-09-12) | L | L | Done this run for prompting-technique keywords; too noisy to vet at this marketplace's scale, no candidate surfaced |

No `decisions/` entries drafted this week — no plugin or token-efficiency technique cleared the
recommend bar. This week's actionable content is entirely `baseline.md` bookkeeping (below) plus
this file.
