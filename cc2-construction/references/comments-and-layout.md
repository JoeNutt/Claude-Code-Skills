# Comments and Layout — detail

## Why the comment rule is this strict

Comments are not free. They are not compiled, not tested, and not checked by anything, so
they drift out of step with the code and become confidently wrong. A stale comment is worse
than no comment, because it is trusted.

The deeper point: **a comment is usually a failure to express something in code.** When you
reach for one, the first question is whether a better name, a smaller function, or a named
constant would carry the meaning instead. Usually it would. That path leaves the explanation
in something the compiler checks and the tests exercise.

This is not an argument for cryptic code with no documentation. It is an argument for putting
the explanation where it cannot rot.

## Docstrings

Write one where the language and project convention expect it — public functions, methods,
classes, modules. Skip it where the name and signature already say everything, and never pad
a file to hit a coverage target for documentation.

**The content, in order:**

1. **What it does** — one or two sentences, in the problem's vocabulary. Describe the
   observable behavior, not the implementation.
2. **Parameters** — name, meaning, and any constraint a caller must satisfy.
3. **Returns** — what comes back and what it means, including the empty or absent case.
4. **Raises / errors** — what can go wrong and under what condition.

**Not in a docstring:** why the function exists, why this approach was chosen, how it works
internally, performance notes, history, alternatives considered, or the author's reasoning.
Those belong in the commit message, the pull request, or a design document — places where
they are dated, attributed, and expected to be historical.

```
"""Apply available account credit to an unpaid invoice.

Args:
    invoice: The unpaid invoice to settle.
    account: The account whose credit is drawn against.

Returns:
    SettlementResult with the amount applied and the remaining balance.

Raises:
    InsufficientCredit: The account has no available credit.
"""
```

Follow the project's existing docstring convention and format — matching what is there
matters more than the exact shape above.

## The self-justification trap

The most common failure in generated code is narrating its own reasoning. It reads as
helpful and is in fact noise: it explains a decision to a reader who can see the code, and it
will be wrong within two refactors.

| Never | Because |
|---|---|
| `# Loop through each item` | The loop is visible |
| `# Using a dict for O(1) lookup` | The data structure is visible; the reasoning is yours, not the code's |
| `# This handles the case where the list is empty` | Either the code shows it or it needs a named guard |
| `# First we validate, then we save` | That is the code, restated |
| `# Increment the counter` | — |
| `# Helper function to format the date` | The name should say this |
| `# TODO: refactor this later` | Untracked and permanent |
| `# Changed by X on 2024-03-01` | Version control's job |
| `# ===== HELPERS =====` | If a file needs signposting it needs splitting |

The distinguishing test: **is the reason outside the code, or inside your head?** A vendor
returning 200 on failure is outside — no reader could deduce it, so record it. "I chose a
dict for speed" is inside — the choice is visible, and the justification is self-commentary.

## Legitimate constraint comments

Short, factual, and pointing outward:

```
# Vendor returns HTTP 200 on failure; body must be checked. SUPPORT-4471.
# Rounding to 2dp before comparison is required by the FCA reporting spec.
# Order matters: the auth token must be set before the region, per SDK issue #812.
# Regex matches the RFC 5322 addr-spec subset the payment provider accepts.
```

Each records something a competent reader could not work out from the code, and each would
cause a real bug if someone "fixed" the code without knowing it. Include the reference —
ticket, spec section, issue number — so the claim can be checked and eventually retired.

## Layout: the newspaper metaphor

A source file should read like a newspaper article. The name at the top tells you whether
you are in the right place. The first section gives you the gist in broad strokes. Detail
increases as you descend, and the fine print is at the bottom.

In practice:

- Public interface and high-level policy near the top.
- Implementation detail below it.
- The lowest-level helpers at the bottom.
- A reader should be able to stop after the first screen with a correct idea of what the file
  does.

## The stepdown rule

Each function should be followed by those at the **next level of abstraction down**, so the
file reads as a continuous descent — every function introducing the ones beneath it.

```
placeOrder()                  <- top level: reads as the story
  ├─ validateOrder()          <- one level down
  ├─ reserveInventory()
  └─ chargePayment()
       ├─ buildPaymentRequest()   <- two levels down
       └─ recordTransaction()
```

The consequence worth internalizing: **one function should mix only one level of
abstraction.** A routine that calls `calculateTotal()` and also does string concatenation and
index arithmetic is operating at two altitudes at once.

The mechanism is worth stating precisely, because it explains why this matters more than it
looks. Every time a reader drops from a high-level line to a low-level one, they have to push
their current train of thought aside to deal with the detail, then pop back to it. **People do
not have a mental stack.** What gets pushed aside is usually just lost, and the reader starts
the paragraph again. A function that alternates levels makes them do this repeatedly — an
abstraction roller coaster.

Symptoms: a routine that both orchestrates named steps *and* zeroes counters, indexes arrays,
or formats strings. Extract the low-level part, name it, and let the top level read as a
sequence of intentions.

## Vertical and horizontal formatting

- **Vertical openness:** blank lines between concepts, none within one.
- **Vertical density:** lines that belong to one thought stay together.
- **Vertical distance:** declare variables close to use; keep a caller and its callee near
  each other; keep conceptually related functions adjacent.
- **Horizontal:** short lines. If a line needs scrolling or wrapping to read, extract part of
  it into a named variable.
- **Indentation** reflects scope depth honestly — never collapse a block onto one line to
  avoid a level.
- **Team style wins.** A consistent style you dislike beats a mix of styles you like. Use the
  project's formatter, and never reformat unrelated code in a change — it buries the real
  edit in noise.
