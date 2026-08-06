# Skills seen

Every marketplace and plugin already evaluated, so the weekly skills scan only surfaces genuinely
new entries instead of re-evaluating the same ones forever. Append, don't rewrite history — if a
verdict changes later (e.g. a rejected skill becomes relevant), add a new dated line rather than
editing the old one.

## Marketplaces

| Date | Marketplace | Source | Status |
|---|---|---|---|
| 2026-08-06 | `claude-plugins-official` | github.com/anthropics/claude-plugins-official | known (pre-existing default) |
| 2026-08-06 | `superpowers-dev` | github.com/obra/superpowers | added this session |
| 2026-08-06 | `wshobson/agents` (`claude-code-workflows`) | github.com/wshobson/agents | found, not added — large (~94 plugins) multi-harness marketplace; individual plugins evaluated below on their own merits rather than adding the whole marketplace speculatively |

## Plugins

| Date | Plugin | Marketplace | Verdict | Reason |
|---|---|---|---|---|
| 2026-08-06 | `superpowers` v6.2.0 | superpowers-dev | **installed** | User explicitly requested it by name; 14 skills covering TDD, systematic debugging, planning, code review, and collaboration patterns directly map to how we work. See `decisions/2026-08-06-install-superpowers.md`. |
| 2026-08-06 | `pensyve` v1.3.0 | wshobson/agents | rejected | Cross-session memory runtime — genuinely addresses a real baseline gap (no cross-project memory system), but ships lifecycle hooks that run automatically with no visible independent security review. Worth revisiting once it has more track record or the hooks' behavior is better documented. |
| 2026-08-06 | `tdd-workflows` v1.3.1 | wshobson/agents | rejected | Conflicts with `test-driven-development`, already installed via superpowers — no reason to run two competing TDD skill implementations. |
