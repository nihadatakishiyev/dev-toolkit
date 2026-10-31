# Weekly research — 2026-10-31

**Deep-dive topic:** Prompting techniques (rotation's 3rd topic — MCP servers & integrations →
agent architectures & multi-agent patterns → **prompting techniques** → memory systems →
testing/verification approaches → tooling/infrastructure/automation → repeats). Previous passes:
2026-08-08 (first pass, confirming verdict), 2026-09-19 (revisit, named "loop engineering" and
"context engineering" framings, confirming verdict again). Last week's pass (2026-10-24) covered
agent architectures & multi-agent patterns.

## Skills scan

Re-added all six marketplaces (ephemeral sandbox, per standing note in `baseline.md`):
`anthropic-plugin-directory` (built-in default), `superpowers-dev`, `claude-code-workflows`,
`impeccable`, `thedotmack`, `claude-community`.

- **`anthropic-plugin-directory` grew 3598 → 3601** (+3) — a much smaller week-over-week change
  than the ~11x restructuring jump two weeks ago (2026-10-10/17 → 2026-10-24). First sign the
  post-restructure count is stabilizing into normal incremental growth rather than continuing to
  balloon.
- `claude-code-workflows` unchanged at 94; `impeccable`/`superpowers-dev`/`thedotmack` unchanged
  (1/1/2 plugins); `claude-community` unchanged at 2283.
- **Narrow, topic-driven keyword sweep** (prompting-technique-specific terms: `prompt-engineer`,
  `chain-of-thought`, `few-shot`, `meta-prompt`, `step-back`, `prompt-optim`, `dspy`,
  `prompt-cach`, `prompt-compress`, `self-consist`, `tree-of-thought`, `structured-output`,
  `jailbreak`, `red-team`, `prompt-injection`) across all six marketplaces matched 22 entries —
  individually read and evaluated (all rejected, see `skills-seen.md` for full reasoning):
  `mackforge-red-team`, `prompt-optimizer`, `goal-composer`, `harness-architect`, `strawman`,
  `llm-dark-patterns` (all `anthropic-plugin-directory`/`claude-community`), `superpowers-optimized`
  (`claude-community`, a modified fork of the already-installed `superpowers` — conflicts directly,
  rejected on that basis alone), plus several more that were generic substring matches on unrelated
  tooling (`redash-mcp`, `slack-mcp`, `web-ctl`, etc. — matched on "prompt" appearing incidentally
  in longer descriptions, not genuinely on-topic).
  - Worth flagging: **`mackforge-red-team`** ("one adversarial pass that assumes your plan or
    answer is wrong, finds the most likely fatal flaw, and names the cheapest check that settles
    it") is thematically exactly the still-unexercised `Workflow`-tool Generator-Verifier/
    adversarial-verify idiom this account already has natively and has flagged unused for five+
    consecutive passes (see `baseline.md`'s Testing/verification section). Its existence as a
    third-party plugin is genuine corroboration that this pattern is in real demand elsewhere, not
    a hypothetical gap — but the plugin itself comes from a personal, apparently auto-generated-
    looking GitHub handle (`dtiger1889-ops`) with no visible reputability signal, so it doesn't
    clear the bar on its own merits. The better fix remains exercising the native tool already
    available, which is a manual-review action item, not something this routine can force.
  - A broader single-word sweep (`prompt`/`context`/`token`/`cach`/`compress`/`reasoning`) against
    `anthropic-plugin-directory` alone matched 406/3601 entries — far too noisy to vet individually
    at this scale, same precedent as prior large-marketplace sweeps (2026-10-10/17/24).
- **New marketplace discovery:** web search found no new *installable* (`.claude-plugin/
  marketplace.json`-based) marketplace beyond the already-known pair plus discovery-directory
  sites. One new discovery-adjacent item: **`skillselion.com`** — markets "thousands of ranked
  skills, loaded into your agent mid-task, no install" via its own MCP server rather than a git
  marketplace `claude plugin marketplace add` can consume. Same non-installable-mechanism category
  as the already-noted `qaskills.sh` (2026-08-22) — noted, not added, no action.

No plugin recommended this week. See `skills-seen.md` for all individual verdicts.

## Token-efficiency / context-management check

**One genuinely new, first-party-sourced finding** — see
`decisions/2026-10-31-token-efficiency-concise-instructions.md` for the full writeup. Summary:
Anthropic's Claude Code team (Thariq Shihipar, posted 2026-07-24, title "The new rules of context
engineering for Claude 5 generation models") reports cutting **more than 80% of Claude Code's own
system prompt** for Claude Opus 5 / Fable 5 with **no measurable loss on coding evals** — the
lesson being that these models had been over-constrained by rigid rules, repeated instructions, and
hardcoded examples, and respond better to concise, judgment-preserving instructions plus
well-designed tool interfaces (e.g. an enum field over a hardcoded usage example in a tool
description) than to exhaustive prose. The primary post wasn't directly fetchable from this
sandbox (`claude.com` redirects to `claude.dev`, both outside this session's allowed network
egress) — corroborated instead across nine+ independent secondary sources that agree on the same
specific numbers and framing, treated as reliable despite not reading the primary text.

This is directly on-topic for this repo's own job (curating `~/.claude/CLAUDE.md` as a deliberately
dense, incrementally-grown instruction set) — drafted as a token-efficiency recommendation with two
required not-applicable boundaries: never apply to an existing hard-boundary instruction (git
safety, the quality-first guardrail itself), and treat as a bias for writing *new* instructions
going forward, not license to retroactively prune existing content without an eval harness this
account doesn't have. See the decision doc for full reasoning on both boundaries.

Secondary coverage also mentions a `/doctor`-style command for rightsizing skills/`CLAUDE.md` —
this sandbox's own installation does have a `claude doctor` command (confirmed by running it), but
its `--help` text describes general installation-health checking; whether it's the exact same
rightsizing feature described in secondary coverage isn't confirmed from here. Worth checking
directly (`claude doctor` or `/doctor` in an interactive session) during the next manual review.

No other new technique found this week that wasn't already reflected in `baseline.md` (deferred
MCP tool-definition loading, context-window compaction behavior, subagent output-limit increases —
all platform-managed internals already either documented or explicitly out of scope per the
design's non-goal on re-litigating harness internals every week).

## Deep-dive: Prompting techniques

Third pass on this topic (2026-08-08 original, 2026-09-19 first revisit, this one the second
revisit). Findings this time:

1. **The ~80% system-prompt-cut finding above is this pass's main addition** — filed under
   token-efficiency (it's squarely a CLAUDE.md-authoring lever) rather than duplicated here, but
   it's also the clearest prompting-technique-shaped finding of the three passes on this topic so
   far: a concrete, first-party-sourced claim about *how* to write instructions for Claude 5
   specifically, not just a confirming restatement of pre-5 guidance.
2. **General web search for prompting-technique guidance this week was thin and mostly
   SEO/blogspam** (see search results: towardsdatascience, Leanpub previews, buildfastwithai,
   gaugr.app, dsstream — practitioner opinion, not primary sources) — restates already-known
   guidance (interview-first/front-load-alignment patterns, rich context, CLAUDE.md as the
   highest-leverage lever, verifiable-task framing with tests-first, XML tags/structure/few-shot
   mattering more for chat than agentic coding, prompt-injection/governance caution). Nothing here
   contradicts or meaningfully extends what's already in `baseline.md`'s prompting-adjacent
   sections.
3. **"Context engineering" framing** (named in the 2026-09-19 pass as the successor framing to
   "prompt engineering") is now even more consistently the dominant framing across this week's
   sources too — reconfirms, doesn't newly establish. One piece of added nuance from this week's
   sources: context engineering is explicitly framed by several sources as *subsuming* prompt
   engineering (the narrower, how-you-phrase-it layer) rather than replacing it — consistent with,
   not a correction to, the 2026-09-19 note.
4. **No contradicting evidence** to the 2026-08-08/2026-09-19 confirming verdict that current
   practice (superpowers' plan/execute/verify skills, this repo's own `specs/` workflow,
   subagent-dispatch patterns, spec-driven development) already substantially tracks 2026 SOTA for
   agentic coding prompting technique.

**Net assessment:** third consecutive confirming pass for this topic's *core* guidance, but unlike
the prior two passes this one surfaced one genuinely new, concrete, actionable, first-party-sourced
technique (the system-prompt-density finding) rather than only vocabulary/framing. Filed as a
token-efficiency decision rather than a separate prompting-technique one since its actionable form
is specifically "how densely to write `CLAUDE.md`/skill instructions," which is this section's
established home for this kind of lever.

## Candidate improvements

| Candidate | Impact | Effort | Status |
|---|---|---|---|
| Add the ~80%-system-prompt-cut finding to `~/.claude/CLAUDE.md`'s token-efficiency section (concise-new-instructions bias, with the two required boundaries) | M | L | Recommendation drafted this run (`decisions/2026-10-31-token-efficiency-concise-instructions.md`), pending manual review |
| Confirm whether this sandbox's `claude doctor` command is the same CLAUDE.md/skill-rightsizing feature described in secondary coverage of the Anthropic post | L | L | Not actionable from this ephemeral sandbox — flagged for next manual-review/persistent-environment session |
| Actually exercise the `Workflow` tool's Generator-Verifier (adversarial-verify) idiom on a real task | M | M | Still open — corroborated again this run by `mackforge-red-team`'s existence as a third-party attempt at the same pattern; this routine can't force the trial itself, next verification-shaped task is the venue |
| Re-confirm the `anthropic-plugin-directory` rename/restructure from a persistent (non-ephemeral) environment | L | L | Still not independently reconfirmed (carried forward from 2026-10-24); this week's +3 growth is consistent with it being real and settling into normal incremental growth, not new counter-evidence either way |
| Revisit `llm-dark-patterns` if it matures (currently brand-new personal-maintainer org, no track record, but a larger-than-usual automatic-hook footprint across nearly every lifecycle event) | L | L | Not actionable yet — noted as a revisit candidate alongside `hol-guard`/`superself` |

No plugin cleared the recommend bar this week (see `skills-seen.md`). One token-efficiency
decision drafted (above). `baseline.md` updated: `anthropic-plugin-directory` count refreshed,
"Weekly research routine" push-confirmation streak extended.
