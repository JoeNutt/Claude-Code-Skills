---
name: sad-user-stories
description: This skill should be used when writing, reviewing, splitting or estimating user stories and their acceptance criteria - for a backlog, a ticket, a sprint, or section 6 of a systems analysis and design document. It carries the fixed story format, the rules for each clause, mandatory Given/When/Then acceptance criteria including the denial case, story splitting patterns, and the anti-patterns that make a story unreviewable. Trigger phrases include "user story", "user stories", "write a story for", "acceptance criteria", "AC", "given when then", "story format", "split this story", "is this story too big", "definition of done", "write up a ticket", "backlog item", "as a user I need".
---

# User stories

A user story states **who needs something, what they need to be able to do, and the outcome
that justifies it**. It is the unit that connects a person's need to the work, and it is the
artifact a reviewer checks completion against.

Within a `sad` document this is section 6. It is used far more often on its own — a ticket, a
backlog item, a sprint's worth of grooming — which is why it is a separate skill.

## The format

```
US-1  As a <job role>, I need to be able to <feature>, to complete <task>.
```

Each clause has a job, and a story fails if any clause does the wrong one:

| Clause | Must be | Must not be |
|---|---|---|
| `<job role>` | A named role that exists in the organisation — intake officer, branch manager, owner | "the user", "the system", "the admin" when no such role was identified |
| `<feature>` | A capability the role exercises | An implementation — a dropdown, a table, an endpoint, a database |
| `<task>` | The outcome that justifies the feature | A restatement of the feature in other words |

The third clause is the one that gets abused. "As a branch manager, I need to be able to view
expenses, to complete viewing expenses" is circular and tells a reviewer nothing. The task is
what becomes impossible if the feature is absent — *to stop a case exceeding its quoted
budget*. If no such consequence can be named, the story is not yet justified and should not be
built.

## Rules

- **One role, one capability, one outcome.** A story needing "and" in any clause is two
  stories. Split it.
- **No implementation in the feature clause.** How it is built is a design decision, made
  later and recorded elsewhere. A story that names a UI control has pre-empted that decision.
- **The role must be able to perceive it.** If no role notices the result, it is not a user
  story — it is a functional or non-functional requirement. Write it as one.
- **Stories are numbered `US-n` and never renumbered.** Requirements, tests and commits refer
  to them by number.
- **A story with no acceptance criteria is not finished.** See below.

## Acceptance criteria are mandatory

One set per story, numbered `AC-n.m`, each independently checkable by someone who did not
write the code:

```
US-4  As a branch manager, I need to be able to see expenses logged against
      each case, to complete a check that no case exceeds its quoted budget.

      AC-4.1  Given a case with logged expenses, when I open the case, then
              the total of all expenses is shown against the quoted value.
      AC-4.2  Given a case whose expenses exceed its quote, when I open the
              case, then it is flagged as over budget.
      AC-4.3  Given a case belonging to another branch, when I request it,
              then access is denied and no case detail is disclosed.
```

**Given** is the starting state. **When** is the single action. **Then** is the observable
result — observable meaning a reviewer can see it without reading the implementation.

**Always write the denial criterion.** `AC-4.3` is the one that gets omitted, and the only one
that proves the control works. Criteria covering the permitted path say nothing about the
forbidden path, and the forbidden path is where the defects are. The same applies to the
empty case, the limit case and the failure case: a story about a list needs criteria for an
empty list and for one too large to return.

## Why this rigour matters with agents

Acceptance criteria are the mechanism that makes "done" checkable rather than declared. A
story reading *"they need to share different documents in different folders for different
cases"* could be satisfied by a shared drive or by a case-management platform, and an
implementer working at volume will choose one and keep going. Criteria that name the states,
the denials and the limits remove that latitude.

Write every story for an implementer who will not ask a clarifying question.

## Splitting a story that is too big

A story that cannot be completed and reviewed as one piece gets split along one of these
seams — never into "part one" and "part two", which produces two halves neither of which
delivers an outcome:

| Seam | Split by |
|---|---|
| Workflow step | Record the enquiry · produce the quote · accept the quote |
| Business rule | Standard-rate quote · high-risk-rate quote |
| Data variation | Single-branch case · case spanning branches |
| Role | What intake does · what the manager approves |
| Path | The happy path · the rejection path |
| Effort | Manual entry now · bulk import later |

Each resulting story must still name a role, a capability and an outcome. If a fragment
cannot, the seam was wrong.

## Anti-patterns

| Written as | Problem | Fix |
|---|---|---|
| "As a user, I need the system to work properly" | No role, no capability, no outcome | Identify who, and what they do |
| "As a developer, I need to refactor the expense module" | No role-perceivable outcome | This is a requirement or a chore, not a story |
| "As an owner, I need a dashboard with charts" | Names the implementation | "...to see which branches are profitable" |
| "As a manager, I need to add and edit and delete expenses" | Three stories | Split on the verb |
| A story with no `Then` a reviewer can observe | Not reviewable, so not completable | Rewrite the criterion around an observable result |
| Stories written after the code | Documents what was built, not what was needed | Write before, review against |
