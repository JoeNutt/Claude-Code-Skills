---
name: cc2-table-driven
description: This skill should be used when replacing a large switch statement, a long if/else chain, or a nested conditional tree with a data lookup, and when mapping inputs to behavior, rates, or configuration. It covers direct-access, indexed-access, and stair-step lookup tables. Trigger phrases include "refactor this switch", "simplify these conditionals", "use a lookup map", "too many if statements", "reduce cyclomatic complexity", "map these cases", "tax brackets", "grading scale", "discount tiers".
---

# Table-Driven Methods

Complex branching logic is usually data wearing a costume. Moving it into a table
collapses cyclomatic complexity, makes new cases additive rather than structural, and
turns "did we handle every branch?" into a question you can answer by reading a table.

## When this applies

Look for the transformation when the code has:

- A `switch` or `if/else` chain over three or more cases that all do structurally the same
  thing with different values.
- Conditionals testing the same expression repeatedly against different constants.
- Nested conditionals selecting a rate, label, fee, message, or handler.
- A chain of range comparisons (`if (x < 10) ... else if (x < 20) ...`).

Do **not** apply it when the branches do genuinely different work with different control
flow, side effects, or early exits. A table of dissimilar closures is harder to read than
the conditional it replaced. Complexity moved is not complexity removed.

## Procedure

1. Identify the **lookup key** — the value being tested — and confirm every branch tests it.
2. Identify what actually varies between branches: a value, a handler, or a range boundary.
3. Choose the access method from the table below.
4. Extract the data to a named, module-level constant, separated from the logic that reads it.
5. Handle the miss case explicitly — an unmatched key must produce a defined result or a
   deliberate error, never a silent `undefined`.
6. Confirm behavior is unchanged. This is a refactor: no new cases, no altered outcomes.

## Choosing an access method

| Method | Use when | Shape |
|---|---|---|
| **Direct access** | The key maps straight onto an index or a unique name | Array or hash map, keyed by the input |
| **Indexed access** | The key space is large, sparse, or non-sequential | A small index map from key → dense-table position, then the data table |
| **Stair-step** | Inputs fall into *ranges*, not exact matches | Ordered list of upper bounds, walked until the input fits |

Direct access is the default; reach for the others only when it does not fit. Full worked
examples of all three, including miss-case handling and the ordering trap in stair-step
tables, are in `references/access-methods.md` — read it when the choice is not obvious or
when implementing indexed or stair-step lookups.

## Keeping the result honest

- Name the table for its contents (`TAX_BRACKETS`, `RETRY_POLICY_BY_STATUS`), not its type.
- Keep table data adjacent to nothing else — no logic interleaved between entries.
- One row per case, in a stable order a reader can scan.
- Where the language has exhaustiveness checking or a sum type, prefer it over a raw map;
  it gives you the missing-case guarantee at compile time.
- The lookup routine stays trivial. If it grows conditionals of its own, the table is
  wrong shape.
