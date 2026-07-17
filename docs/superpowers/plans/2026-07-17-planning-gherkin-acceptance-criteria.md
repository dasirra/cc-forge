# Planning Gherkin Acceptance Criteria Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace `/forge:planning`'s prose bullet-list acceptance criteria with Given/When/Then scenarios, written to stay strictly at PM altitude via two new planner rules (reachability, no illustrative literals) and enforced by four new critic attack vectors, with no code fences, no executable-Gherkin syntax, and no change to `/forge:building` or the epic template.

**Architecture:** Documentation-only change, entirely inside `skills/planning/SKILL.md`: the Phase 3 planner brief, the Phase 4 critic brief, and the child issue template's "Acceptance criteria" section. No build step, no test runner. Each task's "test" is a grep-verified before/after check, plus a final full read-through for coherence, matching the verification approach used in the sibling plan for `skills/building/SKILL.md`.

**Tech Stack:** Markdown only. No dependencies, no build tooling.

## Global Constraints

- Never use the em dash symbol anywhere written; restructure the sentence instead (user's global writing rule). Before finalizing, grep the diff for `—` and `–`.
- No fenced code blocks, `Feature:` headers, or `Scenario:` headers anywhere in the new Acceptance criteria format: bolded **Given**/**When**/**Then** on plain markdown lines only, per the approved design.
- Preserve the "User story" line above the Acceptance criteria section exactly as-is; this plan does not touch it.
- Match the existing prose style of `skills/planning/SKILL.md`: the planner and critic briefs are written as blockquoted (`>`) role prompts; new bullets must use the same `> - ` list-item style as their neighbors.
- **No version bump, no CHANGELOG.md entry, in this plan.** PR #12 (from the sibling plan) is still open and already bumped `package.json` / `.claude-plugin/plugin.json` to 2.2.0; bumping again here from the current unmerged 2.1.1 base would conflict. The user will bump the version themselves once, after both PRs have landed.

---

## Task 1: Update the child issue template's Acceptance criteria format

**Files:**
- Modify: `skills/planning/SKILL.md` (the `### Child body template` section)

**Interfaces:**
- Produces: the rendered Given/When/Then format every planner-authored issue will use, which Tasks 2 and 3 instruct the planner and critic to produce and attack, respectively.

- [ ] **Step 1: Confirm the current template text**

Run:
```bash
grep -n "^## Acceptance criteria" -A 2 skills/planning/SKILL.md
```
Expected output (current state, prose bullet list):
```
## Acceptance criteria
- <observable behavior, human-readable>
...
```

- [ ] **Step 2: Edit the template**

Use the Edit tool on `skills/planning/SKILL.md` with:

old_string:
```
## Acceptance criteria
- <observable behavior, human-readable>
...
```

new_string:
```
## Acceptance criteria
**Given** <precondition>
**When** <trigger>
**Then** <outcome>

**Given** <precondition>
**When** <trigger>
**Then** <outcome>
...
```

- [ ] **Step 3: Verify the edit landed**

Run:
```bash
grep -n "^\*\*Given\*\* <precondition>" skills/planning/SKILL.md
```
Expected: two matches (both scenario placeholders in the template).

Run:
```bash
grep -c '```' skills/planning/SKILL.md
```
Expected: same count as before this edit (this change must not add or remove any code fence: confirm by comparing to the pre-edit file if in doubt, or simply confirm the new template text you just added contains no triple-backtick).

- [ ] **Step 4: Commit**

```bash
git add skills/planning/SKILL.md
git commit -m "docs(planning): render acceptance criteria as Given/When/Then"
```

---

## Task 2: Add the two planner rules (reachability, no illustrative literals)

**Files:**
- Modify: `skills/planning/SKILL.md` (Phase 3, "Planner drafts")

**Interfaces:**
- Consumes: the Given/When/Then format from Task 1 (the rules describe how to fill it in).
- Produces: the two named rules Task 3's critic attack vectors serve as the adversarial backstop for.

- [ ] **Step 1: Confirm the current Phase 3 planner brief text**

Run:
```bash
grep -n "Rules: no technical details of any kind" skills/planning/SKILL.md
```
Expected: one match.

- [ ] **Step 2: Edit the planner brief**

Use the Edit tool on `skills/planning/SKILL.md` with:

old_string:
```
> - 3-8 sub-issues, each with: a title, a user story ("As a <user>, I want
>   <capability>, so that <outcome>"), a proposed-behavior description
>   covering empty/error/edge states, human-readable acceptance criteria
>   (observable behavior, no implementation), and out-of-scope notes.
> - Ordered phases, where each phase leaves something shippable and
>   demonstrable. Record blocked-by / blocks between issues.
>
> Rules: no technical details of any kind. You define what the product should
> do and in what order, nothing about how. If the feature genuinely needs only
> 1-2 issues, say so; do not manufacture an epic.
```

new_string:
```
> - 3-8 sub-issues, each with: a title, a user story ("As a <user>, I want
>   <capability>, so that <outcome>"), a proposed-behavior description
>   covering empty/error/edge states, acceptance criteria written as
>   Given/When/Then scenarios (observable behavior, no implementation), and
>   out-of-scope notes.
> - Ordered phases, where each phase leaves something shippable and
>   demonstrable. Record blocked-by / blocks between issues.
>
> Rules: no technical details of any kind. You define what the product should
> do and in what order, nothing about how. If the feature genuinely needs only
> 1-2 issues, say so; do not manufacture an epic.
>
> Each acceptance criterion is one Given/When/Then scenario: bolded
> **Given**, **When**, **Then** on their own lines, no code fence, no
> Feature/Scenario headers. A Given is valid only if a tester could put the
> product into that state using nothing but the product itself, never a
> database, config file, or other internal state, even when it doesn't sound
> technical (for example, "Given the session token has expired" is not
> reachable this way). A concrete number or string may appear only if that
> value is itself the requirement (for example, "shorter than 8 characters"),
> never as a stand-in example whose exact value doesn't matter (for example,
> "a cart with 3 items").
```

- [ ] **Step 3: Verify the edit landed**

Run:
```bash
grep -n "A Given is valid only if a tester could put the" skills/planning/SKILL.md
```
Expected: one match.

Run:
```bash
grep -c "—" skills/planning/SKILL.md
```
Expected: `0`.

- [ ] **Step 4: Commit**

```bash
git add skills/planning/SKILL.md
git commit -m "docs(planning): add reachability and no-illustrative-literal rules for the planner"
```

---

## Task 3: Add the four critic attack vectors and extend the diagram/story mismatch attack

**Files:**
- Modify: `skills/planning/SKILL.md` (Phase 4, "Critic attacks")

**Interfaces:**
- Consumes: the Given/When/Then format from Task 1 and the two planner rules from Task 2 (these attacks are the adversarial check on whether the planner actually followed them).

- [ ] **Step 1: Confirm the current Phase 4 critic brief text**

Run:
```bash
grep -n "Unverifiable acceptance criteria" skills/planning/SKILL.md
```
Expected: one match.

- [ ] **Step 2: Edit the critic brief**

Use the Edit tool on `skills/planning/SKILL.md` with:

old_string:
```
> - Missing scope: user needs or edge cases the stories silently drop.
> - Unverifiable acceptance criteria: anything a reviewer could not check by
>   using the product ("works well", "is intuitive").
> - Diagram/story mismatch: flows that skip error paths, states with no way
>   out, diagrams that contradict the acceptance criteria.
> - Phase ordering: phases that are not shippable alone, dependencies that are
>   wrong or missing, hidden coupling between issues.
```

new_string:
```
> - Missing scope: user needs or edge cases the stories silently drop.
> - Unverifiable acceptance criteria: anything a reviewer could not check by
>   using the product ("works well", "is intuitive").
> - Unreachable Given: a precondition a user could not establish or observe
>   through the product itself, including one that does not sound technical.
> - Smuggled action: an action disguised as a precondition (for example,
>   "Given the user has submitted the form"), or a When that bundles more
>   than one user action.
> - Vacuous Then: an outcome that merely restates the trigger succeeding, or
>   one not observable from the user's own vantage point.
> - Illustrative literal: concrete data whose value is not itself the
>   requirement being specified.
> - Diagram/story mismatch: flows that skip error paths, states with no way
>   out, diagrams that contradict the acceptance criteria, or a Given
>   describing a state the Proposed behavior or a diagram says cannot occur.
> - Phase ordering: phases that are not shippable alone, dependencies that are
>   wrong or missing, hidden coupling between issues.
```

- [ ] **Step 3: Verify the edit landed**

Run:
```bash
grep -n "Unreachable Given\|Smuggled action\|Vacuous Then\|Illustrative literal" skills/planning/SKILL.md
```
Expected: four matches, one per new attack.

Run:
```bash
grep -c "—" skills/planning/SKILL.md
```
Expected: `0`.

- [ ] **Step 4: Commit**

```bash
git add skills/planning/SKILL.md
git commit -m "docs(planning): add Gherkin-specific attack vectors to the critic brief"
```

---

## Task 4: Full read-through for coherence

**Files:**
- Read only: `skills/planning/SKILL.md` (no modification expected; this task is a verification gate, not an editing task)

**Interfaces:**
- Consumes: the combined output of Tasks 1-3.

- [ ] **Step 1: Read the whole file top to bottom**

Run:
```bash
cat -n skills/planning/SKILL.md
```

Check, in order:
1. The child template's Given/When/Then format (Task 1) uses the exact same
   bolded-keyword style the Phase 3 planner rule (Task 2) describes, no
   drift like `**GIVEN**` vs `**Given**`.
2. The Phase 4 critic attacks (Task 3) reference concepts consistent with
   Task 2's rules: "Unreachable Given" maps to the reachability rule,
   "Illustrative literal" maps to the no-illustrative-literals rule, using
   the same terms.
3. No leftover placeholder text ("TBD", "TODO") was introduced by any edit.
4. Zero em dash characters (`grep -c "—" skills/planning/SKILL.md` returns
   `0`) and zero en dash characters (`grep -c "–" skills/planning/SKILL.md`
   returns `0`).
5. No fenced code block, `Feature:` header, or `Scenario:` header was
   introduced anywhere in the Acceptance criteria section or the two
   updated briefs.
6. The "User story" line in the child template is unchanged from before
   this plan.

- [ ] **Step 2: If any inconsistency is found, fix it directly with the Edit tool, re-run the Step 1 read, and confirm it is resolved**

(No code sample here: this step only fires conditionally, and the fix depends
on what Step 1 finds. If Step 1 finds nothing, skip straight to Step 3.)

- [ ] **Step 3: Commit only if Step 2 made a fix**

```bash
git add skills/planning/SKILL.md
git commit -m "docs(planning): fix coherence issue found in full read-through"
```

If Step 2 made no fix, there is nothing to commit; this is the final task in
the plan.

---

## Self-Review Notes

**Spec coverage:** Task 1 covers the spec's Section 1 (format). Task 2 covers
Section 2 (planner rules). Task 3 covers Section 3 (critic attack vectors).
Section 4 (compatibility with `/forge:building`) required no code change per
the spec itself ("no change to `/forge:building` is needed"), so no task
implements it; Task 4 confirms nothing in the diff accidentally implies
otherwise. The spec's Out of Scope section (epic template, `/forge:interview`,
retroactive migration, real Gherkin syntax, a mechanical check step) required
no task by design. The version-bump omission is a deliberate, user-confirmed
deviation from the sibling plan's pattern, recorded in Global Constraints
above rather than silently dropped.

**Placeholder scan:** No TBD/TODO introduced; every new sentence is complete,
concrete text ready to commit as-is.

**Type consistency:** The term "Given/When/Then" and the bolded keyword style
are identical everywhere they appear (Task 1's template, Task 2's planner
rule, Task 3's critic attacks). The two rule names used informally in this
plan ("reachability rule", "no illustrative literals") are not literal
strings that need to match anywhere in the shipped text; only the concepts
need to match, which Task 4 Step 1 checks explicitly.
