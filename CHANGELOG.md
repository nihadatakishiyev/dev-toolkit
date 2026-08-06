# Changelog

Running index of changes actually implemented (not just researched/proposed). Newest first, each
entry links to its full `decisions/` record.

## 2026-08-06

- **Installed `superpowers` plugin** (v6.2.0, user scope) — see
  [`decisions/2026-08-06-install-superpowers.md`](decisions/2026-08-06-install-superpowers.md).
- **Created this repo** — passive changelog/decision-log for how Claude's own setup evolves over
  time, across every project. See
  [`decisions/2026-08-06-toolkit-created.md`](decisions/2026-08-06-toolkit-created.md) and the full
  design at
  [`specs/2026-08-06-continuous-self-improvement-design.md`](specs/2026-08-06-continuous-self-improvement-design.md).
- **Set skills auto-install policy** (install new, review-before-retire old) — see
  [`decisions/2026-08-06-skills-autonomy.md`](decisions/2026-08-06-skills-autonomy.md).
- **Created and ran the weekly research routine for the first time.** Confirmed the routine's cloud
  sandbox is ephemeral, so its `claude plugin install` calls don't persist — corrected the skills
  lane to *recommend* rather than install in-session. See
  [`decisions/2026-08-06-fix-skills-autonomy-ephemeral-sandbox.md`](decisions/2026-08-06-fix-skills-autonomy-ephemeral-sandbox.md).
  First deep-dive (MCP servers & integrations) written manually from the same verified findings
  after the automated run hit a separate write-access blocker (see `baseline.md` known gaps) — see
  [`research/2026-08-06-mcp-servers-integrations.md`](research/2026-08-06-mcp-servers-integrations.md).
- **Evaluated 2 skills from a newly found marketplace** (`wshobson/agents`) — both declined
  (`pensyve`: unreviewed auto-running hooks; `tdd-workflows`: conflicts with superpowers). See
  `skills-seen.md`.
- **Second weekly research pass: agent architectures/multi-agent patterns + first full per-plugin
  skills scan of `claude-plugins-official` and `wshobson/agents`.** Recommended 3 plugins for the
  next manual review to install (not installed in-session — see the ephemeral-sandbox decision
  above): `playwright` (browser E2E, closes a gap already named in `baseline.md`), `block-no-verify`
  (enforces the existing no-bypass-flags git rule as a hook), and `claude-code-setup` (automates
  new-project tooling analysis). ~24 other plugins evaluated and rejected with reasons in
  `skills-seen.md`. Added `baseline.md`'s missing "Agent architecture / multi-agent tooling"
  section. See
  [`research/2026-08-06-agent-architectures-multi-agent-patterns.md`](research/2026-08-06-agent-architectures-multi-agent-patterns.md),
  [`decisions/2026-08-06-install-playwright.md`](decisions/2026-08-06-install-playwright.md),
  [`decisions/2026-08-06-install-block-no-verify.md`](decisions/2026-08-06-install-block-no-verify.md),
  and
  [`decisions/2026-08-06-install-claude-code-setup.md`](decisions/2026-08-06-install-claude-code-setup.md).
- **Write-access blocker resolved** — the GitHub App was granted access to all repos; the routine's
  third run (its second research pass) pushed successfully on the first try, confirmed live.
- **Installed the 3 recommended plugins** (`playwright`, `block-no-verify`, `claude-code-setup`) —
  manual review confirmed the routine's reasoning and installed all three in this persistent
  environment. See
  [`decisions/2026-08-06-manual-review-installs-three-recommended.md`](decisions/2026-08-06-manual-review-installs-three-recommended.md).
- **Broadened scope beyond skills** (user request) — the weekly routine now also does a
  token-efficiency check every run, not just a rotation topic, and the design's framing is now
  explicitly "state of the art AI-assisted software development," not skills-centric.
- **Created `~/.claude/CLAUDE.md`** — the missing mechanism for a *behavioral* decision (not just a
  plugin install) to actually change future sessions. First entries: explicit Haiku routing for
  cheap/high-volume subagent work, memory-system discipline. See
  [`decisions/2026-08-06-global-claude-md.md`](decisions/2026-08-06-global-claude-md.md) and
  [`research/2026-08-06-token-efficiency-and-agent-landscape.md`](research/2026-08-06-token-efficiency-and-agent-landscape.md).
- **Added an explicit quality-first boundary to the token-efficiency section** — user feedback:
  don't trade quality for token savings. Every efficiency instruction now has to state when it
  doesn't apply, not just when it does; enforced as a standing requirement for future entries too,
  not just today's two. See
  [`decisions/2026-08-06-quality-first-guardrail.md`](decisions/2026-08-06-quality-first-guardrail.md).
- **Design/UI-UX deep-dive pass (user request), installed 3 plugins.** Researched SOTA UI/UX and
  product design practices for AI-assisted development; grounded the "generic AI aesthetics"
  finding against Compass's own unstyled `create-next-app` scaffold. Installed
  `frontend-design@claude-plugins-official` (official, counters generic-AI-aesthetics), and
  `modern-web-guidance@claude-plugins-official` (Google Chrome, 130+ web-platform/accessibility
  checks) directly. Added a new marketplace (`impeccable`, github.com/pbakaus/impeccable, 56k★) and
  installed its single companion plugin after reading its auto-running hook script to confirm it's
  purely local/no-network before install. Evaluated and declined Figma/design-handoff MCP (no
  current use case). Added a "UI/UX & product design" standing-instructions section to
  `~/.claude/CLAUDE.md` (accessibility floor, screenshot self-critique loop, heuristic review pass)
  so the installs actually get used, not just sit there. See
  [`research/2026-08-06-design-ux.md`](research/2026-08-06-design-ux.md),
  [`decisions/2026-08-06-install-design-plugins.md`](decisions/2026-08-06-install-design-plugins.md),
  and
  [`decisions/2026-08-06-design-workflow-standing-instructions.md`](decisions/2026-08-06-design-workflow-standing-instructions.md).
