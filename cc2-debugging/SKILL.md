---
name: cc2-debugging
description: This skill should be used when diagnosing a defect, an exception, a failing test, or unexpected behavior in existing code - anything where the cause is not yet known. It applies the scientific method of debugging: stabilize the error, find the minimal reliable reproduction, form and test hypotheses, fix the root cause, verify, then look for the same defect elsewhere. Trigger phrases include "why is this failing", "this test breaks", "debug this", "it works locally", "this is flaky", "unexpected output", "getting an error", "find the bug", "it crashes when", "this is broken".
---

# Debugging

Debugging is not editing code until the symptom disappears. It is finding out **why** the
program does what it does, and the difference shows up months later in whether the defect
comes back.

The governing rule: **understand the problem before changing anything.** A fix you cannot
explain is a guess, and a guess that makes the symptom vanish is worse than no fix — it
converts a visible defect into a hidden one.

## The five steps

### 1. Stabilize the error

An intermittent defect cannot be diagnosed. Before anything else, make it happen on demand.

- Find the **minimal reliable reproduction**: the smallest input, state, and sequence that
  still produces the failure, every time.
- Narrow by bisection — halve the input, halve the code path, halve the commit range.
- If it will not stabilize, you are missing a variable. Suspect timing, ordering,
  concurrency, uninitialized state, environment, locale, clock, randomness, cached data,
  or test pollution from a previous case.
- A test that reproduces it is worth more than a note describing it — write it first, watch
  it fail, and it becomes the verification step later for free.

Do not skip this because the cause "seems obvious". The obvious cause is right often enough
to be dangerous and wrong often enough to cost a day.

### 2. Locate the source

This is the scientific method, applied literally:

1. **Gather data** through repeated experiment — not by staring at the code and imagining.
2. **Form a hypothesis** that accounts for *all* the data, not just the convenient parts.
3. **Design an experiment** that proves or disproves it — ideally one whose two outcomes
   point in genuinely different directions.
4. **Run it**, and record what happened.
5. **Repeat** until the hypothesis survives an attempt to disprove it.

Practical technique:

- Vary one thing at a time. Two simultaneous changes make the result uninterpretable.
- Keep a written record of what you have ruled out. Debugging failure is usually
  re-testing the same hypothesis three times, not running out of ideas.
- Narrow by bisection in space as well as time: which module, which routine, which line.
- Use the tools rather than guessing — a debugger, a stack trace, `git bisect`, a diff
  against the last working version, logging at the boundary, a profiler for performance.
- Suspect the code that changed most recently, the code you wrote most recently, and the
  code with the weakest tests — in that order.
- Prefer a narrow, well-chosen experiment over broad instrumentation. Print statements
  scattered everywhere produce volume, not evidence.

### 3. Fix the defect

- Fix the **cause**, not the symptom. If you cannot explain why the change works, you have
  not found the cause yet — go back to step 2.
- Understand the problem before you change anything, and understand the program beyond the
  immediately failing line. A local patch to code you do not understand tends to relocate
  the defect rather than remove it.
- Change one thing at a time, so the fix's effect is attributable.
- Save the original. Keep the working tree recoverable so a failed fix costs nothing.
- Add or fix the test that should have caught this, and make sure it fails before your
  change and passes after.
- Resist "make the symptom go away" edits: a swallowed exception, a null check that hides a
  broken invariant, a retry around a race, a widened type, a loosened assertion. Each buys
  quiet now and pays for it later.

### 4. Test the fix

- Verify against the reproduction from step 1.
- Re-run the surrounding tests, not just the one that failed — fixes introduce defects at a
  meaningful rate, and the code around a defect is the likeliest place for the next one.
- Check the boundaries of whatever you changed.

### 5. Look for similar errors

The step almost everyone skips, and the one with the best return.

Before closing it out, ask: **where else does this same mistake live?** The same
copy-pasted block, the same misunderstood API, the same missing guard, the same wrong
assumption by the same author on the same afternoon. Grep for the pattern, not the symptom.
Then ask what would have prevented the whole class — a type, a lint rule, a barricade, a
named constant.

## Anti-patterns

These are the reliable ways to waste a day:

- **Guess-and-check.** Scattering changes to see what helps. It corrupts the code, and each
  edit adds a variable to an experiment that already had too many.
- **Superstitious fixing.** "It works now" without a causal account. Blaming the compiler,
  the library, or the machine before your own code — occasionally right, almost always
  premature, and never the first hypothesis worth testing.
- **Fixing the symptom.** Special-casing the failing input rather than correcting the logic
  that mishandles it.
- **Not reproducing first.** Verifying a fix against a defect you could never reliably
  trigger proves nothing at all.
- **Ignoring contradicting data.** A hypothesis that explains four of five observations is
  wrong. The fifth is the interesting one.
- **Not reading the error.** The message, the stack trace, and the line number are evidence,
  and they are frequently read past on the way to a theory formed before looking.

## Reporting

When you have found it, say plainly: what the defect was, why it produced this symptom,
what the fix changes, and where else the pattern appears. If you have *not* found it, say
what you ruled out and what you would test next — never present a symptom's disappearance
as a diagnosis.

Harder cases — concurrency, heisenbugs, performance, memory, and defects that only appear in
production — are covered in `references/hard-cases.md`.
