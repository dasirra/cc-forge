# Design: Given/When/Then acceptance criteria in /forge:planning

Date: 2026-07-17
Status: approved design, pending spec review

## Problem

`/forge:planning`'s child issues carry an "Acceptance criteria" section that
is today a flat bullet list of prose: `- <observable behavior,
human-readable>`. These issues are the text `/forge:building` bases its
contract negotiation on, so how standardized and concrete they are at
planning time matters downstream. Robert "Uncle Bob" Martin has publicly
described a personal workflow of Gherkin (Given/When/Then) into acceptance
tests into unit tests into code, which prompted the question of whether
forge should adopt Gherkin somewhere in its pipeline. A companion design
(`docs/specs/2026-07-17-building-contract-examples-and-gate-design.md`)
already answered this for `/forge:building`: Gherkin's shape, not its
executable tooling, in exactly one field (a criterion's worked `example`).
This design answers the separate question of whether and how Gherkin fits
`/forge:planning`, which sits at PM altitude and has different constraints
than `/forge:building`'s DEV altitude.

## Why this is harder than the building-skill case

`/forge:planning` is forbidden from naming files, schemas, functions,
libraries, tables, or endpoints; its acceptance criteria describe observable
product behavior only. Gherkin's `Given` clause is, in most of its real-world
use (Cucumber, SpecFlow, behave), where technical fixture setup lives
("Given the users table has 3 rows"), so adopting the format naively risks
reintroducing exactly the kind of technical leak the altitude discipline
exists to prevent, including leaks that don't read as obviously technical
("Given the session token has expired" sounds like plain English but
describes system state a user cannot directly establish or observe).
Gherkin also risks freezing concrete example data before the codebase is
understood, the same premature-freezing concern the building-skill design
raised, and adopting Gherkin's literal syntax (`Feature:`, `Scenario:`, code
fences) invites the expectation of executable tooling that forge does not
want anywhere in its planning stage, since these issues are read by a human
on GitHub, never executed.

None of these risks are fatal. Each has a specific, checkable guardrail; the
sections below adopt Gherkin's three-clause shape while closing each one.

## Goal

Replace `/forge:planning`'s prose bullet list of acceptance criteria with
Given/When/Then scenarios, one per criterion, written to stay strictly at PM
altitude, with guardrails a planner can follow and a critic can mechanically
attack.

## Design

### 1. Format

The child issue template's "Acceptance criteria" section changes from:

```
## Acceptance criteria
- <observable behavior, human-readable>
...
```

to one Given/When/Then trio per criterion, as plain bolded-label markdown
lines, no code fence and no `Feature:`/`Scenario:` headers:

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

Worked examples (the format actually reviewed and approved):

> **Given** a user with an empty shopping cart
> **When** they open the checkout page
> **Then** they see a message inviting them to add items, and no checkout
> button is shown

> **Given** a user creating a new password
> **When** they enter a password shorter than 8 characters
> **Then** they see an error stating the minimum length, and the password
> field is not accepted

No fenced code block: a fence signals "machine artifact" and pulls toward
Cucumber idioms (data tables, step definitions) that add nothing here, since
nothing executes these scenarios, and it would also break normal markdown
rendering (links, emphasis) inside the block for a human reader on GitHub.
Bolded keywords are kept (rather than folding into a single unlabeled
sentence) because labeled clauses are what let the critic, and the human
reviewer, attack a scenario clause-by-clause: "this criterion's Given is
unreachable" is a precise objection only if the Given is visibly delimited.

The "User story" line above this section ("As a `<user>`, I want
`<capability>`, so that `<outcome>`") is unchanged.

### 2. Two new planner rules

Added to the Phase 3 planner brief in `skills/planning/SKILL.md`, as
explicit, checkable rules rather than left to the generic "no technical
content" instruction:

- **Reachability rule**: a `Given` is valid only if a tester could put the
  product into that state using nothing but the product itself (clicking
  around, not touching a database, config file, or internal system state).
  This catches both obvious leaks ("Given the database contains no rows for
  this user") and disguised ones that don't read as technical vocabulary
  ("Given the session token has expired").
- **No illustrative literals**: a concrete number or string may appear in a
  scenario only if that value is itself the requirement being specified
  ("a password shorter than 8 characters" is fine because 8 is the policy),
  never as a stand-in example whose exact value doesn't matter ("a cart with
  3 items" is not fine, since the requirement holds for any non-empty cart).

### 3. Four new critic attack vectors

Added to the Phase 4 critic brief's attack list in `skills/planning/SKILL.md`,
as the adversarial backstop for the two planner rules above (mirroring the
building-skill design's two-layer pattern: instruct, then attack):

- **Unreachable Given**: a precondition a user could not establish or
  observe through the product itself. The critic's enforcement of the
  reachability rule.
- **Smuggled action**: an action disguised as a precondition (e.g., "Given
  the user has submitted the form" describes a trigger, not a starting
  state), or a `When` that bundles more than one user action. Either makes
  the scenario untestable as a single check.
- **Vacuous Then**: an outcome that merely restates the trigger succeeding
  ("When the user adds an item, Then the item is added") rather than
  describing an actual observable consequence, or an outcome not observable
  from the user's own vantage point.
- **Illustrative literal**: concrete data whose value is not itself the
  requirement. The critic's enforcement of the no-illustrative-literals
  rule.

One existing attack gets a small extension: "diagram/story mismatch" also
now covers a scenario whose `Given` describes a state the issue's Proposed
Behavior section or a diagram says cannot occur.

### 4. Compatibility with /forge:building

No change to `/forge:building` is needed for it to consume the new format.
Its Phase 0 already treats issue bodies as prose to summarize ("Summarize the
user stories and acceptance criteria you extract"); Given/When/Then-labeled
lines are still plain markdown text to a subagent reading them. This is
recorded here as confirmed compatibility, not an assumption left untested.

## Out of scope

- Any change to the epic body template. It has no acceptance-criteria
  section of its own (Key decisions and Phased delivery only), so nothing
  there changes.
- Any change to `/forge:interview`'s spec template. Its "User stories"
  section is an earlier, purely narrative artifact, not per-criterion, and
  is unaffected.
- Retroactive migration of existing GitHub issues already filed under the
  old bullet-list format. This only changes what new planning runs produce
  going forward.
- Real Gherkin/Cucumber syntax (`Feature:`, `Scenario:` headers, fenced code
  blocks, data tables) anywhere in `/forge:planning`. Rejected for the
  reasons in Section 1: it invites an executable-tooling expectation this
  stage does not want, and degrades plain markdown rendering for human
  reviewers.
- A mechanical (non-adversarial) check equivalent to the building skill's
  "lead verifies grounding mechanically" step. There is no lead-side
  grep-equivalent for prose reachability the way there is for a `file:line`
  grounding claim, so enforcement here is two-layer (planner rule, critic
  attack), not three-layer.

## Open questions

None outstanding; the format, planner rules, and critic attack vectors were
each reviewed and approved above.
