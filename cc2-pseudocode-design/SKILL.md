---
name: cc2-pseudocode-design
description: This skill should be used before implementing a routine or module whose logic is non-trivial - complex algorithms, multi-branch business rules, state machines, or anything where the approach should be agreed before code is written. It produces a pseudocode outline with inputs, outputs, preconditions and postconditions for review first, so the design is verified before any syntax is generated. Trigger phrases include "design this first", "how should I approach", "plan this function", "before you write it", "outline the logic", "walk me through the approach", "pseudocode", "what's the algorithm".
---

# Pseudocode Programming Process

For non-trivial logic, design the routine in plain language **before** writing code. Errors
in intent are far cheaper to fix in a sentence than in a syntactically valid implementation,
and reviewing an outline is faster than reviewing an implementation of the wrong idea.

## When to use this

Use it when the logic is genuinely non-obvious:

- Multi-branch business rules, state machines, non-trivial algorithms.
- Anything where getting the approach wrong means rewriting rather than editing.
- Anything the user has signalled they want to agree before implementation.

**Skip it** when the routine is obvious. Outlining a three-line getter is ceremony, and
ceremony trains people to skim the outlines that matter. If the implementation is shorter
than its own outline, just write it.

## The process

### 1. Check the prerequisites

Before designing, be able to state:

- **Purpose** — what the routine is for, in one sentence, in problem-domain terms.
- **Inputs** — each parameter, its type, its valid range, and who guarantees that.
- **Outputs** — the return value and its meaning, including for every failure case.
- **Preconditions** — what must be true on entry. Whose job is it to ensure this: the
  caller's, or a barricade upstream?
- **Postconditions** — what is true on exit. What has changed, what has been allocated,
  what invariant now holds.
- **Side effects** — I/O, mutation, state, anything the signature does not reveal. If the
  list is long, that is a design finding, not a footnote.
- **Failure modes** — what can go wrong, and whether each is an error path or an assertion.

If any of these cannot be stated, that is the actual problem. Resolve it before designing —
an outline built on an unstated precondition just relocates the ambiguity.

### 2. Write the outline in intent, not syntax

Write each step as a statement of *what* and *why*, in plain English, at a level a reviewer
could check without knowing the language.

```
# Purpose: settle an invoice against available credit, partially if necessary.
# In:  invoice (unpaid, positive amount), account (exists, active)
# Out: SettlementResult - amount applied, remaining balance
# Pre:  caller has validated the invoice belongs to the account
# Post: account credit reduced by exactly the amount applied; invoice unchanged on failure

determine how much credit is actually available on the account
if no credit is available
    return a zero settlement, leaving the invoice untouched
work out the settleable amount: the lesser of the invoice total and available credit
reserve that amount against the account, failing if another settlement got there first
record the settlement so it can be reconciled later
return what was applied and what remains outstanding
```

Rules for the outline:

- Describe intent, not mechanics. `reserve that amount, failing if another got there first`
  states a design decision; `call reserve()` states nothing.
- Stay language-agnostic. No syntax, no library calls, no types beyond the header.
- One level of abstraction throughout. A step reading "increment the loop counter" is
  written at the wrong altitude.
- Each step should be roughly one line of comment to a few lines of eventual code. A step
  that will expand into thirty lines is a routine of its own — name it and outline it
  separately.
- If a step is hard to phrase, the design is not settled yet. That difficulty is the point
  of doing this.

### 3. Review before implementing

**Present the outline and stop.** Do not continue into implementation in the same turn
unless the user has said to. The entire value is the checkpoint.

Check it yourself first: does it handle the failure modes listed in step 1, hold one
abstraction level, satisfy the postconditions on every path including error paths, and stay
within the construction limits (one job, three parameters or fewer, complexity under 10)?

Iterate on the outline while it is still cheap.

### 4. Implement

Once the outline is agreed:

- Fill in the code beneath each step.
- **The outline is scaffolding, not comments.** Delete it as the code replaces it. An outline
  line that restated what the code now shows is exactly the noise the comment rule forbids.
  The header block becomes the docstring — what it does, params, returns, raises — and
  nothing else survives unless it records a genuine external constraint.
- If implementing reveals the outline was wrong, fix the outline first, then the code. Do
  not let the implementation silently diverge from the reviewed design.

## Output format

Give the header block (purpose, in, out, pre, post) followed by the numbered or indented
steps, then a single explicit question — whether to proceed, or which alternative to take
where you saw more than one reasonable design.
