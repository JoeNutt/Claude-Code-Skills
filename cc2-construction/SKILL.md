---
name: cc2-construction
description: This skill should be used whenever writing, editing, reviewing, or refactoring code in any programming language. It carries the user's baseline software construction standards - naming, routine and class structure, parameter limits, control flow and nesting, data scope, comments, and refactoring discipline - and applies to all code work by default, not only when quality is explicitly mentioned. Trigger phrases include "write", "implement", "add a function", "create a class", "fix", "refactor", "clean up", "review this code", "rename", "extract", and any request that produces or changes source code.
---

# How I Write Software

Language-agnostic construction standards, derived from Code Complete 2. Apply them to every
line you write or review, in every project, for the remainder of the task - not just to the
snippet that triggered this skill.

Precedence: the target language's idiom wins over any convention here, and a project's own
`CLAUDE.md` wins over both. Where nothing conflicts, these are the default.

The governing idea: **manage complexity**. Code is read far more often than it is written,
so read-time clarity always outranks write-time convenience.

## Naming

- Names are the primary documentation. Vague identifiers (`temp`, `data`, `flag`, `info`,
  `obj`, `x`) are only acceptable as short-lived loop indices.
- Procedures take a strong verb-noun pair: `calculateLoanPayment()`, not `doPayment()`.
- Functions are named for what they return: `isEndOfStream()`, `customerCount`.
- Booleans must read as a predicate — `isFound`, `hasPermission`, `canRetry`. Never name a
  boolean negatively (`notFound`, `isNotValid`); it produces double negatives at the call site.
- No magic numbers or magic strings. Extract literals to named constants at first use.
- Follow the target language's idiom for case and ordering. Idiom wins over any convention here.

## Routines

- One routine, one job. A name needing "and" or "or" (`initAndProcess`) means it must be split.
- Maximum seven parameters. Beyond that, pass a structured options/config object.
- Order parameters input → modify → output, consistently across the whole codebase.
- Keep cyclomatic complexity under 10 per routine. Past that, extract.
- Avoid boolean parameters that select behavior; `render(true)` is unreadable at the call
  site and usually wants to be two routines.

## Classes and abstractions

- Design around abstractions, not raw data. A class models a thing in the problem domain;
  its interface should read as operations on that thing.
- Hold one consistent level of abstraction per interface. Do not mix high-level business
  operations with low-level data manipulation in the same public surface.
- Hide information, especially the parts most likely to change. Ask "what does this class
  conceal?" — if nothing, it is a data bag, not an abstraction.
- Maximum seven data members before the class is a decomposition candidate.
- Default to the most private visibility the language offers. Expose only what the
  abstraction genuinely requires — never make a routine public merely because it happens
  to call other public routines.
- Prefer containment ("has a") over inheritance. Use inheritance only for genuine "is a"
  substitutability, and keep hierarchies at most three levels deep.
- Keep coupling loose: talk to immediate collaborators, not to objects returned by other
  objects (`a.getB().getC().doThing()` is a coupling smell), and never write code that
  depends on another component's internals beyond its public contract.

Deeper design guidance — information hiding, isolating likely changes, the coupling
criteria, and how to proceed when the design is not obvious — is in
`references/design-heuristics.md`. Read it when designing a new module, splitting a class,
or when the right structure is genuinely unclear rather than merely unwritten.

## Data and scope

- Declare and initialize a variable immediately before its first use, never at the top of a
  long block. Minimize the live span between first and last reference.
- Keep scope as narrow as the language permits.
- Avoid global mutable state. Where it is unavoidable, mediate every access through
  dedicated accessor routines rather than direct mutation.
- Prefer integers to floats where rounding error accumulates (money, counters, iteration).

## Control flow

- Write the nominal path first, then the unusual and error cases. The main purpose of a
  block should be visible without scrolling.
- Structure logic as sequence, selection, and iteration. Make execution dependencies
  explicit — pass one step's output as the next step's argument rather than relying on
  ordering the reader has to infer.
- Use early returns for guard clauses at the top of a routine, then keep a single exit for
  the main path. Multiple returns are justified only where they genuinely beat nesting;
  scattered returns through a long routine are not.
- Nesting deeper than three levels is a defect. Extract the interior into a named routine.
- Simplify negated compound conditions with DeMorgan's laws:
  `if (!a || !b)` becomes `if (!(a && b))`.
- Use a counted loop only when the iteration count is known up front; otherwise use a
  conditional loop. Avoid loops that break arbitrarily from the middle.
- A sprawling `if/else` chain or `switch` over data is usually a lookup table in disguise.
- No `goto`. Restrict recursion to genuinely hierarchical data, keep it within a single
  routine, and never build cyclic recursion chains across routines — they are close to
  impossible to trace.

## Comments and layout

- Code explains *how*; comments explain *why* — the business rule, the trade-off, the
  workaround and its cause. Never write a comment that restates the syntax.
- Delete commented-out code; version control already holds it.
- Separate logical "paragraphs" of code with blank lines so visual structure mirrors
  logical structure.

## Working discipline

- **Refactoring changes structure, never observable behavior.** When asked to refactor, do
  not add features or alter business logic in the same pass.
- Do not optimize before the code is correct, complete, and tested. Tune only against
  measurements, never a hunch — ask for a benchmark rather than guessing at a hot path.
- Cover boundary values, equivalence partitions, and failure paths — not just the happy path.
- Validate untrusted input once, at the boundary. Do not re-validate deep in internal logic.
- Assertions are for conditions that must never happen (programmer error). Error handling is
  for conditions expected in production (bad input, network failure). Never confuse the two.
- Be honest about limits: never invent an API, flag, or signature you have not verified, and
  say plainly when a proposed design has a real problem rather than agreeing with it.
- Notice repeated toil and offer to automate it.
