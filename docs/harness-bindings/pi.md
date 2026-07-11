# Pi binding

Forge's commands (`commands/interview.md`, `commands/planning.md`,
`commands/building.md`) are written in harness-neutral prose: instead of naming
a specific coding agent's tools or models, they use neutral verbs and tier
markers that any harness can bind to its own equivalents. This doc is the Pi
binding: it maps every neutral phrase used in the prose to the Pi coding
agent's concrete tool or, for tiers, the open-weight model that realizes it.
The sibling Claude Code binding is
[`docs/harness-bindings/claude-code.md`](claude-code.md).

| Neutral phrase (in commands/) | Pi binding |
| --- | --- |
| judgment-tier | GLM-5.2 |
| labor-tier | GLM-5.2 |
| an interactive question | a plain chat question to the user, options listed with the recommended one first |
| the worktree helper | `git worktree add` |
| exploratory | a read-only subagent (the `subagent` tool from `pi-subagents`, forked context) |
| a working scratch location | a `harness/` directory inside the worktree, git-excluded (per the building command's `.git/info/exclude` step) |

Read each neutral phrase in the prose as the Pi tool, model, or behaviour its
right-hand value describes, and realise that behaviour. Unlike the Claude Code
binding — whose round-trip check needs an exact-string mapping — this is a
semantic binding the Pi agent interprets, not a literal find-and-replace.
Several right-hand values are descriptions rather than drop-in tokens (for
example `exploratory` and `an interactive question`): when the prose says "an
exploratory subagent", spawn a read-only forked-context subagent; do not paste
the description into the sentence.

## Single model, both tiers

Unlike the Claude binding (opus for judgment, sonnet for labor), Pi runs
GLM-5.2 for every role. The judgment/labor distinction stays legible in the
prose but does not drive model selection here — there is no cheaper tier to
fall back to. This is not a regression: the adversarial pairs were already
single-model on Claude Code (planner and critic both opus; generator and
evaluator both opus), so what makes a subagent an adversary was never the
weights, it is the separated context.

Note the consequence, though. With the same model on both sides of every pair,
**context separation is the only thing preventing a rubber-stamp**. On Claude
Code a stronger judgment model gave some slack; here it does not. The
`exploratory`/subagent binding below is therefore load-bearing, and the
open-model reliability compensations — built once issue #4's validation shows
which are actually needed — matter more.

## Required: forked-context subagents

Forge's adversaries — planner vs critic, generator vs evaluator — only work
because each cannot see the other's reasoning. Pi core ships no subagent tool;
the `pi-subagents` package provides one with forked contexts. **Do not run
forge on a Pi without it.** Without forked contexts the pipeline collapses into
self-review, the exact failure the design exists to prevent, and no amount of
prose fixes that. Do not simulate a subagent by continuing in the same context;
if the package is absent, stop and say so rather than faking the separation.

## Evaluation surfaces (not yet neutralized)

The `web` / `native` / browser-MCP lines in `commands/building.md` were left
Claude-specific — neutralizing the evaluation-surface table was out of scope
for the command neutralization. On Pi, read them as follows until that
follow-up lands:

- `native` (computer-use) surface: **unavailable**. Skip any step whose only
  observation path is native, tell the user, and continue.
- `web`: drive the browser with **Playwright via Bash**, not a browser MCP.
- `library` / `cli` / `service`: need only Bash; these port unchanged.

## Not bound

There is no review / `/code-review` entry — that step is absent from the
command prose (it was removed from the pipeline upstream). If you want a final
review on Pi, spawn a reviewer subagent over `git diff <base>...HEAD` with a
brief to find correctness and cross-workstream defects.
