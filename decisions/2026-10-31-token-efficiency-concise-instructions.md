# Decision: recommend a "write new instructions concisely, don't over-constrain" note for the token-efficiency section

**Date:** 2026-10-31 (drafted by the weekly research routine — recommendation, not yet applied to
`~/.claude/CLAUDE.md`)

## Context

This week's prompting-techniques deep-dive turned up a first-party Anthropic finding (Thariq
Shihipar, Claude Code team, posted to `claude.com/blog/the-new-rules-of-context-engineering-for-
claude-5-generation-models`, dated 2026-07-24 per secondary coverage — the primary post itself
wasn't fetchable from this sandbox, both `claude.com` and its `claude.dev` redirect target are
outside this session's allowed network egress; the finding is corroborated across nine+
independent secondary write-ups that agree on the same specifics, so treated as reliable despite
not reading the primary source directly): the Claude Code team removed **more than 80% of Claude
Code's own system prompt** for Claude Opus 5 / Fable 5 (the Claude 5 generation), with **no
measurable loss on coding evals**. The stated lesson: these models had been over-constrained by
rigid rules, repeated instructions, and hardcoded examples in the system prompt, `CLAUDE.md`
files, and skills; the better approach for Claude 5-generation models is to state things once,
lean on well-designed tool interfaces (e.g. an enum field over a hardcoded usage example), and
trust the model's own judgment instead of over-specifying. Secondary coverage also mentions a
`/doctor`-style command for rightsizing skills/`CLAUDE.md` size — this account's own Claude Code
installation does have a `claude doctor` command (confirmed by running it; its own `--help` text
describes "health of your Claude Code installation," which may or may not be the exact same
rightsizing feature described in secondary coverage — not independently confirmed from this
sandbox, worth a direct check during manual review).

This is squarely on-topic for this repo's own job: `~/.claude/CLAUDE.md` is explicitly maintained
here as a deliberately-curated "compiled output" of behavioral decisions (see
`decisions/2026-08-06-global-claude-md.md`), and every one of this routine's own prior
recommendations has added to it incrementally, bullet by bullet, without a standing instruction
about *how densely* those bullets should be written. If the finding above is directionally
correct, an ever-growing list of rigid, example-laden bullets is itself a quiet tax on the exact
quality/cost tradeoff this section exists to manage — not because any single bullet is wrong, but
because the cumulative density works against the model's own judgment rather than with it.

## Decision

Recommend adding a short standing note to `~/.claude/CLAUDE.md`'s token-efficiency section:
**when adding or revising an instruction in `CLAUDE.md` (global or project-level) or a skill, favor
stating the judgment-relevant fact or constraint once, concisely, over a rigid rule plus a worked
example** — e.g. naming the condition that should change behavior, rather than spelling out a
step-by-step procedure for every case already covered by the model's own general competence.

**Required boundary — when this does NOT apply:**
- **Never apply this to an existing instruction whose job is to enforce a hard safety or process
  boundary** (this harness's own git-safety protocol, the "never trade quality for token savings"
  guardrail this very section is built around, the quality-first boundary from
  `decisions/2026-08-06-quality-first-guardrail.md`). Anthropic's own finding was about cutting
  *redundant, over-constraining* material from a system prompt that had accreted it over time — it
  was not a claim that hard boundaries are safe to compress. Trimming a load-bearing rule to save
  tokens is exactly the failure mode this section was created to prevent, not an application of
  this finding.
- **This is a bias for writing *new* instructions going forward, not a mandate to retroactively
  prune what's already in `~/.claude/CLAUDE.md`.** Anthropic's "no measurable loss" claim rests on
  their own internal coding-eval suite; this account has no equivalent eval harness to verify a
  cut doesn't quietly regress something (see the long-standing, still-open testing/verification gap
  in `baseline.md` — no `claude plugin eval`-style measurement has ever been run against this
  account's own `CLAUDE.md`). Without that verification capability, aggressively stripping existing
  content on the strength of someone else's eval numbers would be applying a cost-driven technique
  without being able to check the quality side of the tradeoff — precisely what the quality-bar
  requirement on this section exists to block. Revisit the retroactive-pruning question only if/when
  an eval-style check against this account's own `CLAUDE.md` becomes available.

## Alternatives considered

- **Apply retroactively now — review and compress existing `~/.claude/CLAUDE.md` bullets
  immediately.** Rejected for this pass: no way to verify no quality loss without an eval harness;
  the finding supports "write new things leaner," not "cut what's there on faith."
- **Treat as a platform-internal detail, not actionable** (the same treatment given to automatic
  prompt caching / harness-level auto-compaction elsewhere in this section's history). Rejected:
  unlike those, this is explicitly a *workflow-level authoring* lesson — how a human/agent should
  write instructions for Claude 5 models — not something the platform does automatically regardless
  of what's written. It's a technique this account's own instruction-writing can apply or not,
  which is exactly the category this check exists to catch.
- **Do nothing until the primary source is actually readable from this sandbox.** Considered, but
  nine-plus independent secondary sources converge on the same specific numbers and framing
  (Thariq Shihipar, Claude Code team, ~80% cut, no measurable eval loss, named examples like
  enum-over-hardcoded-example) — treated as sufficiently corroborated to act on with a recommendation
  rather than wait indefinitely for a network exception.

## Reasoning

Same shape as every prior token-efficiency recommendation in this file: a real, newly-surfaced,
first-party-sourced lever, recommended with an explicit and specific not-applicable boundary so it
can't drift into "always cut corners on instructions" — the exact failure mode user feedback
already flagged once for this section (see `decisions/2026-08-06-quality-first-guardrail.md`).
Distinctive here: the boundary isn't generic "don't sacrifice quality" boilerplate, it's two
concrete, checkable carve-outs (hard-boundary instructions are off-limits; retroactive pruning is
blocked pending an eval capability this account doesn't have yet) tied directly to why the
technique could otherwise misfire for this specific account's situation.
