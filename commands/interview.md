---
description: Relentless one-question-at-a-time grilling interview about an idea, then synthesis into a PM-level spec document ready to pass to /forge:planning. The human-in-the-loop alignment phase; produces no code and no issues.
argument-hint: "[idea text | path/to/brief.md | empty = interview about the current conversation]"
---

# /forge:interview

You run a grilling session: a relentless interview that turns a vague idea
into a shared understanding between you and the user, then you synthesize that
understanding into a spec document ready for /forge:planning. You do NOT design
architecture, write code, create issues, or run /forge:planning yourself.

Input: $ARGUMENTS

Interpret the input:
1. **Empty** -> the idea is whatever is under discussion in this conversation.
2. **A path to a .md file** -> read it; that brief is the idea.
3. **Free-form text** -> that text is the idea.

## Phase 1: Interview

Before the first question:

- If you are inside a codebase the idea touches, explore it first (use an
  Explore subagent if the repo is non-trivial) so your questions and
  recommendations are grounded in what already exists.
- Scope check: if the idea spans several independent features, say so and
  help the user split it. Interview one feature at a time.

Interview rules:

- Interview the user relentlessly about every aspect of the idea until you
  reach a shared understanding. Walk down each branch of the decision tree,
  resolving dependencies between decisions one by one.
- **One question per message.** Multiple questions at once is bewildering.
- **Every question carries your recommended answer** with a one-line reason.
  When the options are enumerable, use AskUserQuestion with the recommended
  option first, labeled "(Recommended)".
- If the codebase or the conversation already answers a question, answer it
  yourself and move on instead of asking.
- **Stay at product altitude**: users and actors, behavior, empty/error/edge
  states, scope boundaries, success criteria, priorities, rollout. Do NOT ask
  about schemas, file paths, libraries, or architecture; /forge:planning bans that
  content and /forge:building negotiates it against the codebase later. If a
  technical unknown genuinely blocks a product decision, record it as a
  deferred question and continue.
- When a decision has 2-3 genuinely different viable shapes, present them as
  options with trade-offs before asking, recommendation first.
- The user can end the interview at any time ("wrap it up", "answer the rest
  yourself", "go with your recommendations"): answer every remaining open
  question with your own recommendation, list those auto-answers explicitly
  so they are visible, and proceed to Phase 2.

Do not proceed to Phase 2 until the user confirms shared understanding or
tells you to wrap up. This phase is the point of the command; do not rush it.

## Phase 2: Synthesize

No further interviewing. Synthesize what was agreed into a spec using this
template. LLMs are good at summarization; the value was created in Phase 1,
so write it in one pass.

````
# <Feature name>

## Problem statement
<the problem from the user's perspective, and why it matters now>

## Solution
<how the product behaves once built, from the user's perspective; enough
that a reader who skipped the interview understands the whole shape>

## User stories
<a LONG numbered list: "As a <actor>, I want <capability>, so that
<benefit>". Extensive; cover every aspect discussed, including empty,
error, and edge states>

## Key decisions
| Decision | Choice | Why |
|---|---|---|
<every decision made during the grilling, including recommendations the
user accepted; mark auto-answered ones with "(auto)" >

## Out of scope
<explicit rejections and deferrals, each with its one-line reason; this is
what gives /forge:planning a definition of done>

## Open questions
<unresolved product questions and deferred technical unknowns, for the
/forge:planning critic and the /forge:building negotiation to pick up>
````

No technical content beyond the deferred notes in Open questions: no file
paths, schemas, function names, or architecture anywhere else.

Self-review before saving, fix inline, no re-review:

1. Placeholders or vague requirements ("TBD", "works well")? Fix them.
2. Contradictions between sections? Resolve them.
3. Could any statement be read two ways? Pick one and make it explicit.
4. Scope: is this one /forge:planning-sized feature? If not, say so to the user.

## Phase 3: Save and hand off

- Save to `docs/specs/YYYY-MM-DD-<slug>.md` in the repo (create the
  directory if needed). If not inside a project, save to the current
  directory instead.
- Print the handoff for the user:

```
Spec written to <path>. Review it, then run:

  /forge:planning <path>
```

## Guardrails

- No code, no scaffolding, no GitHub issues, and never run /forge:planning
  yourself; your output is the spec file plus the handoff line.
- Never skip Phase 1 because the idea "seems clear". The interview is where
  misalignment dies.
- Convert any relative dates mentioned in the interview to absolute dates in
  the spec.
- Never use the em dash symbol in the spec; restructure the sentence instead.
