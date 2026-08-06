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
