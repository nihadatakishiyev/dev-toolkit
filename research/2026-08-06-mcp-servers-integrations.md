# Weekly research — 2026-08-06

**Deep-dive topic:** MCP servers & integrations (1st in rotation — no prior deep-dive entries existed)

## Skills scan

New marketplace found: **`wshobson/agents`** (github.com/wshobson/agents, "claude-code-workflows")
— a large, actively maintained multi-harness plugin marketplace (~94 plugins spanning testing,
infra, security, memory, and more; also targets Codex CLI, Cursor, and others besides Claude Code).
Verified real via direct fetch of its `marketplace.json`, not just search snippets. Not added to
`skills-seen.md` as a blanket "watch this marketplace" — individual plugins evaluated on their own
merits below, consistent with the design's per-plugin criteria rather than marketplace-level
trust.

Two candidate plugins evaluated, both **not installed** (see `skills-seen.md` for the authoritative
record):
- **`pensyve`** (cross-session memory runtime) — directly addresses a real gap already noted in
  `baseline.md` ("no cross-project memory or preference system"). Held back specifically because it
  ships lifecycle hooks that run automatically, with no independent security review visible. This
  is exactly the kind of thing worth revisiting in a later pass rather than a permanent no.
- **`tdd-workflows`** — straightforward reject, duplicates `test-driven-development` already
  installed via `superpowers`.

## Deep-dive: MCP servers & integrations

- **Governance shift**: Anthropic donated MCP to the Linux Foundation's newly formed **Agentic AI
  Foundation (AAIF)** in December 2025, alongside Block's `goose` and OpenAI's `AGENTS.md` —
  confirmed via the Linux Foundation's own announcement, not just secondary coverage. MCP is now a
  vendor-neutral, community-governed standard rather than an Anthropic-only spec.
- **Adoption scale**: over 10,000 public MCP servers reportedly in production as of mid-2026,
  adopted beyond Anthropic's own ecosystem (OpenAI, Google DeepMind, Microsoft cited as adopters).
- **Spec evolution**: the 2026-07-28 spec release added a formal Feature Lifecycle and Deprecation
  Policy; a proposed "MCP Server Cards" standard (server metadata via `.well-known` URLs) is on the
  2026 roadmap for discoverability without connecting first.
- **Security**: enough real-world usage now exists that an "OWASP MCP Top 10"-style security
  concern list is circulating — worth a closer look in a future pass specifically, not fully
  chased down this run.

## Compared against `baseline.md`

Our MCP setup (Supabase, Wallet, Gmail, Google Calendar/Drive) is entirely platform-connector-based,
not local `.mcp.json`. Nothing in this pass changes that assessment — no evidence the connector
model is behind the curve, just that the underlying protocol itself has matured/formalized
significantly since baseline was written. No `baseline.md` update needed from this finding alone.

## A real design gap this run surfaced

The scheduled routine's cloud session runs in a genuinely ephemeral sandbox — `claude plugin
marketplace list` showed nothing configured at the start of the run (not because the real setup
changed, but because that specific execution had never seen `~/.claude/plugins` before). This means
**any skill the routine installs only exists for the lifetime of that single run** — it does not
persist to the user's actual environment. The auto-install carve-out in the design spec assumed
"installed" means durably installed; in practice, for a scheduled cloud routine specifically, it
currently only means "installed, logged, and then discarded when the sandbox tears down." This
needs a real fix (see `decisions/` / a future pass) — likely: the routine should treat a
newly-approved skill as a **candidate for the human's own environment** to install (log it clearly
enough that the next manual review picks it up and runs `claude plugin install` in a persistent
environment), rather than actually running the install command itself in a session where it can't
stick. Flagged as high-impact, low-effort to fix — mostly a matter of correcting what "installed"
means in the routine's own instructions.

## Candidate improvements

| Candidate | Impact | Effort |
|---|---|---|
| Fix the auto-install carve-out to log-as-candidate instead of actually running `claude plugin install` inside the (ephemeral) routine session | H | L |
| Revisit `pensyve` once its hooks have more track record / a visible security review | M | L |
| Look into MCP Server Cards (`.well-known` discovery) once it's further along — not actionable yet, spec proposal stage | L | — |
| Do a focused pass on the emerging MCP security guidance (OWASP-style Top 10) — not chased down this run | M | M |
