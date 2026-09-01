# Construction Audit Checklist

Thresholds are decision triggers, not laws. A breach means *look closely and justify*, not
*automatically a defect*. Record the justification when the code is right and the rule is
wrong — that distinction is most of the value of an audit.

---

## 1. Complexity

| Check | Threshold | Smell |
|---|---|---|
| Nesting depth | > 3 levels | Arrow-shaped indentation drifting right |
| Routine length | Doesn't fit on a screen | Needing to scroll to see one routine's logic |
| Decision points | > 10 branches in one routine | Long `&&`/`\|\|` chains, many `if`s |
| Conditional tree over one value | > 3 cases | Candidate for a lookup table |
| Cyclomatic complexity | > 10 per routine | Many independent paths through one routine |
| Inheritance depth | > 3 levels | Behavior scattered up a tall hierarchy |
| Boolean expression terms | > 2-3 terms | Unnamed compound condition |

Ask: what is the deepest, hairiest routine here, and would a new maintainer understand it in
one read? Extract-and-name is the fix for nearly everything in this section: a compound
condition becomes a named predicate, a loop interior becomes a routine.

## 2. Routines and classes

- Does the name describe everything the routine does? A name needing "and"/"or", or a vague
  name (`process`, `handle`, `manage`, `doWork`), signals mixed responsibilities.
- Parameters: more than three without real justification (seven is an absolute ceiling), or
  ordered inconsistently with the rest of the codebase. Arguments that travel together want
  to be a type.
- Boolean parameters that select behavior — `render(true)` is unreadable at the call site
  and usually means two routines.
- Output parameters where a return value would do.
- Class data members: more than seven suggests decomposition.
- Visibility: members public that no external caller uses.
- Coupling: `a.getB().getC().doThing()` chains; a class reaching through another's internals.
- Cohesion: does everything in this class serve one abstraction, or is it a utility bag?
- Does the class hide a source of change, or leak it to every caller? If it hides nothing,
  it is a data bag, not an abstraction.
- Consistent abstraction level: does one public interface mix business operations with
  low-level data manipulation?
- Inheritance used where containment would do — no genuine "is a" substitutability, or a
  subclass overriding inherited behavior to neuter it.
- Routines public only because they call other public routines.
- Semantic coupling: does anything depend on another module's internal behavior rather than
  its interface — call ordering, a side effect, an undocumented guarantee?
- Recursion spanning more than one routine (cyclic recursion chains).

## 3. Naming

- Vague: `temp`, `data`, `info`, `obj`, `val`, `result`, `flag`, `x` outside a loop index.
- Booleans that don't read as predicates, or that are negated (`notFound`, `isDisabled` used
  as `if (!isDisabled)`).
- Procedures without a strong verb-noun pair.
- Functions not named for what they return.
- Magic numbers and magic strings — especially repeated ones, and *especially* the same
  literal appearing in two files.
- Names that lie: the code no longer does what the name says.
- Inconsistent vocabulary: `fetch`/`get`/`retrieve`/`load` used interchangeably for one concept.

## 4. Data and scope

- Live span: declaration far from first use; a variable alive across a long block.
- Scope broader than needed — module-level where function-level would do.
- Global mutable state, and direct mutation of it from many sites.
- Variables reused for two different purposes within one routine.
- Floats where rounding accumulates: money, counters, iterative sums, equality comparisons.
- Collections passed around and mutated by several routines with no clear owner.

## 5. Defensive boundaries

- Where does untrusted data enter? Enumerate every entry point: handlers, deserialization,
  config and environment loading, file reads, datastore reads, public library entry points.
- Is validation consolidated at a boundary, or scattered through interior logic?
- Does the boundary output domain types, or pass raw strings inward?
- Redundant interior checks that the boundary already guarantees.
- Assertions used on user-triggerable input — wrong tool, and absent in release builds.
- Assertions containing side effects — the work vanishes when assertions are compiled out.
- Swallowed exceptions: empty catch blocks, or catches that log and continue as if fine.
- Silent defaults: `catch { return [] }` turning a failure into plausible data.
- Errors carrying secrets, tokens, or personal data into logs or responses.
- Mixed correctness/robustness policy within one subsystem, with no stated reason.

## 6. Comments and layout

- Any comment that is not a docstring or a recorded external constraint. Narration of the
  code, self-justification of a design choice, banners, change logs, author tags.
- Docstrings carrying rationale, implementation notes, or history rather than what / params /
  returns / raises.
- An external constraint that is *not* recorded where a reader could not possibly infer it —
  a vendor quirk, a spec requirement, an ordering dependency imposed from outside.
- Stale comments contradicting the code — worse than none, because they are trusted.
- Commented-out code.
- Missing blank-line separation between logical paragraphs.
- File does not descend from abstract to detailed; helpers above the code that calls them.
- A routine mixing two levels of abstraction.
- Formatting inconsistent with the rest of the file or the project's formatter.

## 7. Testability

- Failure paths, boundaries, and equivalence partitions untested — happy path only.
- Range logic (brackets, tiers, bands) without tests at the exact boundaries and one unit
  either side.
- Routines requiring heavy setup to test, indicating hidden dependencies that should be
  parameters.
- Non-determinism: direct use of clock, randomness, filesystem, or network with no seam.
- Tests asserting on incidental detail (log text, formatting) rather than behavior.

## 8. What this audit cannot see

No single detection technique finds most defects — reading, testing, and review each catch a
different and only partly overlapping set. A clean audit is therefore evidence, not proof.

Say so when it matters. Specifically, flag where review alone is insufficient and something
else is needed:

- Logic that is correct on inspection but has no test proving it stays that way.
- Concurrency and ordering, which reading reliably fails to catch.
- Performance claims, which need measurement rather than an opinion about a hot path.
- Integration behavior between modules that were each audited alone.
- Anything depending on production data shape or scale.

Where a defect class needs a different technique, name the technique — "this needs a test at
the bracket boundaries", "this needs a race detector" — rather than reporting a vague
concern. Prefer specific, checkable findings over impressions: a finding an author can
verify or refute in a minute is worth more than a general unease about a file.

---

## Ranking findings

Order by cost of leaving it, not effort to fix:

1. **Correctness risk** — swallowed errors, silent defaults, boundary and off-by-one defects,
   unvalidated untrusted input, money in floats.
2. **Maintenance hazard** — deep nesting, mixed responsibilities, tight coupling, global
   mutable state, lying or stale names and comments.
3. **Readability friction** — vague names, magic numbers, comment noise, layout that does not
   descend from abstract to detailed.

Findings without a nameable consequence are preferences. Drop them.

## Before reporting

- Have you read every file you report on, in full?
- Does each finding cite a real `file:line`?
- Have you checked whether a project `CLAUDE.md` sanctions what you are flagging?
- Are you flagging language or framework idiom as a defect?
- Could you name the two or three themes behind the findings, and the one change you would
  make first?
