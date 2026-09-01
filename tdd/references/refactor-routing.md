# Refactor Routing and Mechanics

The table in `SKILL.md` is the fast path — run it every cycle. This file is the escalation:
read it when a trigger fired and the right move isn't obvious, or when the refactor looks
bigger than one cycle.

## Cost model

Loading every standards skill on every green would be enormously wasteful — most cycles need
none of them. The tiers:

1. **The trigger table in `SKILL.md`.** Already in context whenever this skill is active.
   Costs nothing extra. Resolves most cycles on its own.
2. **This file.** One read, only when a trigger fired and the fix isn't obvious.
3. **The specific standards skill.** Loaded only for the row that actually fired — one skill,
   not fifteen.

The discipline that makes this cheap: **check the table every cycle, load nothing unless a
row fires.** A cycle that fires nothing should cost one scan and one line of output.

## Expanded triggers

### Duplication — the primary target

Beck's refactor step is specifically about eliminating the duplication created while getting
to green. Look for all of it, not just copy-paste:

- **Identical or near-identical code** in two places.
- **Structural duplication** — the same shape with different types or names.
- **Knowledge duplication** — the same rule, constant, or format expressed in two places.
  A validation rule in the model and again in the handler is duplication even with no shared
  text, and it is the kind that rots: the two copies drift.
- **Test/production duplication** — a magic value repeated in both.
- **Duplication between test cases** — repeated setup wanting a fixture or builder.

The rule of three is a reasonable guide for *extracting* an abstraction, but exact
duplication created in the green step goes now, not on its third appearance.

**One qualification, and it matters.** "Remove all duplication" holds *within* a boundary —
a function, a module, a service. It does not automatically hold *across* independently
developed modules or services: forcing one canonical representation across a boundary couples
the two sides together, and that coupling usually costs more than the duplication did.
Deduplicate within a boundary; translate across one. If the duplicate lives on the other side
of a module or service boundary, that is an `architecture` question, not a refactor.

### Structure
- A routine that needs a comment to explain its middle — extract that part and name it.
- A name containing "and", "or", "manager", "helper", "util", "process", "handle".
- Parameter lists growing past ~5, or a boolean parameter selecting behavior.
- A conditional expression with more than two or three terms — name it as a predicate.
- Nesting from accumulated guard conditions — invert to early returns.
- A class whose fields split into two groups used by different methods — two classes.

### Signals the *design* is off, not the code
- The test needed extensive mocking → too many collaborators, or the wrong ones.
- The test needed the clock, filesystem, network, or randomness → missing seam.
- Setup is longer than the assertion → the unit is doing too much.
- The test asserts on internals → behavior isn't exposed at the right level.
- A private method you want to test directly → it probably wants to be its own unit.

These are design findings. Fixing them is a legitimate refactor, and usually a more valuable
one than tidying names.

## Common refactorings

Take one at a time; run the tests between each.

| Refactoring | When | Move |
|---|---|---|
| **Extract function** | A block needs a comment, or repeats | Name the block, call it |
| **Inline function** | The body is clearer than the name | Replace calls with the body |
| **Extract variable** | A subexpression is unclear | Name the intermediate result |
| **Rename** | The name misleads or is vague | Rename everywhere, at once |
| **Replace magic literal** | A bare number or string | Named constant at first use |
| **Introduce parameter object** | Params travel together | One structured argument |
| **Replace conditional with lookup** | 3+ cases over one value | See `cc2-table-driven` |
| **Decompose conditional** | Complex `if` test | Extract to a named predicate |
| **Replace nested conditional with guards** | Arrow-shaped code | Early return on each edge case |
| **Extract class** | Two responsibilities in one | Split along the field/method groups |
| **Move function** | It uses another class's data more than its own | Move it there |

## When the refactor is too big for the cycle

Do not skip it, and do not smuggle it into the current cycle:

1. Finish the current cycle — green, minimal cleanup, commit.
2. Record what the larger refactor is and why it's needed, in the terms of whichever standard
   it violates.
3. Do it as its own change, starting from green, with no behavior change, committed separately.
4. If it needs the user's decision — an architectural boundary moving, an interface others
   depend on — surface it rather than deciding unilaterally.

A refactor mixed into a feature change is unreviewable, because nobody can tell which edits
were meant to change behavior and which weren't.

## What is not refactoring

The step has a strict definition: **structure changes, observable behavior does not.**

Not refactoring, and not allowed in this step:
- Adding functionality or handling a new case — that is the next RED.
- Fixing a bug you noticed — that is a new failing test first.
- Changing an interface others depend on, without saying so.
- Optimizing for performance. That is code tuning: it comes after correctness, needs a
  measurement, and is not part of this cycle.
- Broad reformatting mixed with structural edits — it hides the real change in the diff.

If the tests go red during a refactor, you changed behavior. Revert and take a smaller step;
do not adjust the tests to match.

## Verdict format

End every cycle with one line, so the step is visibly done rather than assumed:

```
Refactored: extracted `calculateDiscountTier` from `processOrder`; the tier chain
            became a lookup table (cc2-table-driven).
```

```
No refactor needed: checked duplication, routine size, nesting, naming, conditionals,
                    input handling — none fired. 6-line function, single responsibility.
```

The second is a legitimate and common outcome. What is not legitimate is producing neither.
