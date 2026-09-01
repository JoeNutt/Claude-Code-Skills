# Design Heuristics

Design is a **heuristic process**, not a deterministic one. There is no procedure that
converts requirements into the correct structure, no way to know in advance that a design is
right, and no single correct answer — only better and worse trade-offs. Two consequences
worth holding onto:

- Expect to try more than one shape. A design you arrived at without considering an
  alternative is a first draft, not a decision.
- Stop when the design is good enough to build the next piece confidently. Designing past
  that point is speculation, and speculative structure is the most expensive kind to remove.

## The primary goal: manage complexity

Every heuristic below exists to keep the amount a person must hold in their head at once
small enough to reason about.

- **Essential complexity** is inherent in the problem. It cannot be removed, only organized.
- **Accidental complexity** is introduced by the solution — indirection nothing needs,
  abstractions with one implementation, cleverness. All of it is optional, and all of it
  is your doing.

When a design feels hard to explain, that is the signal. A structure you cannot describe in
a few sentences will not be understood by whoever maintains it.

## Information hiding

The most valuable single heuristic. Each module keeps a secret — a data representation, an
algorithm, a protocol, a business rule — and exposes only an interface.

Ask of every module: **what does this hide?** If the answer is "nothing", it is a data bag
or a namespace, not an abstraction, and it will not protect callers from anything.

The related question is: **what would a caller have to know to misuse this?** Anything on
that list should be behind the interface, not in front of it.

## Identify and isolate areas likely to change

Design so that the things most likely to change are the things easiest to change. In
practice:

1. Identify what is volatile — business rules, hardware and OS dependencies, input/output
   formats, third-party APIs, anything a regulator or a vendor controls, anything the
   product manager has opinions about.
2. Separate it from what is stable.
3. Isolate it behind an interface that hides the volatility, so a change to it does not
   ripple outward.

Common volatile areas worth isolating by default: data formats and schemas, external
service clients, configuration, anything time- or locale-dependent, and any rule expressed
in the requirements as a number.

A useful inversion: if a foreseeable change would require edits in more than one or two
places, the design has not isolated it.

## Keep coupling loose

Coupling measures how strongly one module depends on another. Judge it by asking how hard
each module would be to understand, test, or reuse *on its own*.

Loose, in roughly descending order of preference:

- **Simple data parameters** — primitives or plain data passed in a signature.
- **Simple object** — a module instantiates a self-contained object.
- **Object parameter** — a module requires another object rather than raw data.

Tight, and worth removing:

- **Semantic coupling** — one module depends on another's *internal behavior*, not its
  interface: relying on undocumented call ordering, on a side effect, on a global being set
  earlier, or on "it happens to return sorted". The worst kind, because it compiles fine and
  breaks silently later.
- **Control coupling** — passing a flag that tells the other module which branch to take.
- **Reaching through** — `a.getB().getC()` chains that bind you to a structure two modules away.

Good coupling is small, visible, flexible, and evident at the call site. Ask: how much does
a caller need to know about this module's insides to use it correctly? The answer should be
"nothing beyond the signature".

## Levels of design

> Levels 1-2 (system and subsystem) are application architecture: dependency direction,
> layering, ports and boundaries. That is the `architecture` skill's territory. This file
> covers levels 3-5 — design within a module.


Work at the right level; conflating them produces the "big ball of mud" where a request
handler also formats currency.

1. **System** — the whole program and its major divisions.
2. **Subsystem / package** — major responsibility areas, and the rules for which may call
   which. Keep the dependency graph acyclic; a cycle between subsystems means they are one
   subsystem pretending to be two.
3. **Class** — interfaces and abstractions within a subsystem.
4. **Routine** — the internals of each class.
5. **Internal routine design** — the logic inside a routine.

When stuck at one level, the problem is often that a decision belongs at a different one.

## Other heuristics worth applying

- **Form consistent abstractions.** Every class should let a caller think in one vocabulary.
- **Encapsulate implementation details.** Information hiding is the goal; encapsulation is
  the enforcement.
- **Inherit only for genuine "is a" substitutability.** Anything a subclass must override to
  neuter is a signal the relationship is containment, not inheritance. Keep hierarchies to
  three levels; deeper ones defeat the comprehension they were meant to aid.
- **Keep modules small and single-purpose**, and name them for their responsibility.
- **Look for common design patterns** — they supply a shared vocabulary and a checked
  solution. Do not force one where the problem does not fit; a misapplied pattern is
  accidental complexity with a respectable name.
- **Aim for strong cohesion.** Everything in a module should serve one purpose.
- **Build for testability.** A design that is hard to test is telling you about its coupling.

## When the design is not obvious

- Try a second and third approach before committing to the first.
- Design in a cheap medium first — a sketch, a paragraph, pseudocode — where changing your
  mind costs nothing.
- Divide and conquer: solve part of it, and let what you learn inform the rest.
- Work top-down when the structure is clear, bottom-up when the details are better
  understood than the whole. Most real design alternates.
- Prototype the risky part *specifically to learn from it*, and be willing to throw it away.
  A prototype kept because it works is a design decision made by accident.
