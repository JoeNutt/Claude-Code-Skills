---
name: tdd
description: This skill should be used whenever writing or changing production code that can be tested - implementing a feature, fixing a bug, adding a method, or extending existing behavior. It enforces the red-green-refactor cycle with a mandatory, non-skippable refactor step checked against the user's construction and security standards. Trigger phrases include "write a test", "TDD", "test first", "implement this", "add this feature", "fix this bug", "make this pass", "red green refactor", and any request to build functionality in a codebase that has tests.
---

# Test-Driven Development

The user works test-first, exclusively. Follow the cycle; do not write production code ahead
of a failing test that demands it.

## The cycle

**RED — write one failing test.**
- One test, for the next small piece of behavior. Not a suite.
- Write no more of the test than is sufficient to fail (not compiling counts as failing).
- **Run it and watch it fail.** A test never observed failing proves nothing — it may be
  asserting nothing, or passing for the wrong reason. Confirm the failure message is the one
  you expect.
- Test behavior through the public interface, not implementation detail. A test coupled to
  internals blocks the refactor step, which is the step that matters.

**GREEN — make it pass, fast and badly.**
- Write no more production code than is sufficient to pass the currently failing test.
- Speed over elegance. Beck's framing is explicit: commit whatever sins are necessary.
  Hardcode, duplicate, take the shortcut.
- Three ways to get there: *fake it* (return the constant, generalize later), *obvious
  implementation* (just write it, when it truly is obvious), *triangulation* (add a second
  test that forces the generalization). Fake it when the design is unclear; obvious
  implementation when it isn't.
- Run the test. It must pass, and every other test must still pass.

**REFACTOR — improve the structure, keep the behavior.**
- **This step is mandatory and has a required output.** See below.
- Refactor **both the new code and the old code it touched**, and refactor the tests too.
- Behavior must not change. No new functionality, no altered logic — that is the next RED.
- Tests stay green throughout. If they go red, you changed behavior: revert and take a
  smaller step.

Commit at green, and again after refactoring. Two clean commits beat one mixed one.

## The refactor gate

The refactor step is skipped more than any other, for two reasons — and both are wrong:

1. **"The code already looks fine."** It does not. You *just* wrote it to pass a test as fast
   as possible; that is the definition of the green step. Code at green is unrefactored by
   construction. Your familiarity with code you wrote sixty seconds ago is not evidence of
   its quality.
2. **"Refactor against what?"** Against the standards below — an explicit, external baseline,
   not a feeling.

**You may not declare a cycle complete without stating a refactor verdict.** One of:

- `Refactored: <what changed and why>` — the change, named.
- `No refactor needed: checked <the triggers below>, none fired.` — the null result, but
  *explicit*, and only after actually running the checklist.

Silence is not an acceptable third option. Run the checklist **before** forming an opinion
about the code, not after.

## Refactor triggers → where to look

Scan this list every cycle. It is a routing table, not the rules themselves. **Load the
named skill only when a row actually fires** — most cycles fire nothing and cost nothing.

| If you see | Load |
|---|---|
| **Duplication of any kind** — the primary target of this step | `cc2-construction` |
| Routine doing more than one thing; name needs "and"; >7 params | `cc2-construction` |
| Nesting >3 deep; cyclomatic complexity >10; long routine | `cc2-construction` |
| `if/else` chain or `switch` over one value, 3+ cases | `cc2-table-driven` |
| Vague names, magic numbers/strings, negated booleans | `cc2-construction` |
| Class >7 members; deep inheritance; leaked internals; a data bag | `cc2-construction` → `references/design-heuristics.md` |
| Validation scattered through the interior; unclear trust boundary | `cc2-defensive-design` |
| Comments restating the code; missing *why* on a non-obvious decision | `cc2-construction` |
| Untrusted input, file paths, uploads, external payloads | `sec-input-validation` |
| Data reaching a query, command, template, markup, URL, or fetch | `sec-injection-defense` |
| A new endpoint, action, or anything needing a permission check | `sec-authz-least-privilege` |
| Passwords, tokens, keys, secrets, randomness, encryption | `sec-crypto-secrets` |
| Manual memory, `unsafe`, or FFI | `sec-memory-safety` |
| Modules being wired together; build or CI touched | `cc2-integration` |

If several fire, take the most costly first. Do not attempt every improvement in one pass —
one refactoring at a time, tests green between each.

`references/refactor-routing.md` holds the expanded trigger list, the mechanics of common
refactorings, and how to handle a refactor too large for the current cycle.

## When it's clean enough

The target, in priority order — earlier rules win:

1. **Passes all the tests.**
2. **Reveals its intention** — a reader can tell what it does and why.
3. **No duplication** — every piece of knowledge stated once.
4. **Fewest elements** — nothing left that isn't earning its place.

Stop there. Refactoring past this point is speculative design, and speculative structure is
the most expensive kind to remove later.

## Nested cycles

Refactoring is continuous, not a phase at the end:

- **Seconds** — the three laws: no production code without a failing test; no more test than
  fails; no more code than passes.
- **Minutes** — red-green-refactor, once per test.
- **Tens of minutes** — as tests get more specific, the code should get more generic. Check
  for production code that is over-fitted to the tests written so far.
- **Hours** — step back to architectural boundaries: are the modules still right, is the
  dependency direction still clean?

A refactor too large for the current cycle does not get skipped — note it, finish the cycle,
and do it as its own change with the tests already green.

## Working rules

- **One failing test at a time.** Never write a second while the first is red.
- **Never write production code without a failing test that requires it.** If asked to add
  code with no test, write the test first, or say why the code is untestable — untestability
  is a design finding.
- **Bug fixes start with a failing test** that reproduces the bug. It proves the diagnosis
  and prevents the regression. See `cc2-debugging` for finding the cause.
- **Never change a test to make it pass.** Fix the code, or fix a test that was wrong — and
  say which you did and why.
- **Never delete or skip a failing test to get green.** Report it instead.
- **Cover the boundaries and the failure paths**, not just the happy path.
- **Refactor the tests too.** Duplicated setup and unclear test names decay the suite that
  everything else depends on.
- If a test is hard to write, the design is telling you something. Listen to it before
  reaching for a mock.
