---
name: cc2-integration
description: This skill should be used when assembling separate modules into a working system, bootstrapping a new project's structure, adding a module that must interoperate with existing ones, or setting up builds, smoke tests and CI. It covers top-down versus bottom-up integration strategy, incremental assembly, the daily build and smoke test, and how construction formality should scale with project size. Trigger phrases include "wire these together", "set up the project", "integrate this", "add this module", "set up CI", "the build", "scaffold this", "how should this be structured", "connect these services".
---

# Integration

Integration is the order in which finished pieces are combined into a working system. The
order is a real design decision: it determines when defects surface, how hard they are to
localize, and how early anything is demonstrable.

## Integrate incrementally, always

Add one component at a time, test, and only then add the next. The alternative — building
everything and assembling it at the end — makes every defect a search across the whole
system, and it is the reason "big bang" integration reliably overruns.

The payoff of incremental assembly: defects are localized to the piece just added, there is
a working system at all times, and progress is observable rather than asserted.

**Rule: never add a second untested component on top of a first.** When something breaks,
the newest piece is the prime suspect, and that inference only holds if there is one.

## Choosing a direction

| Strategy | Build order | Needs | Best when |
|---|---|---|---|
| **Top-down** | Skeleton and high-level control first, stub the layers beneath | Stubs | The architecture is the risk; you want a demonstrable skeleton early |
| **Bottom-up** | Utilities and leaf classes first, assemble upward | Test drivers | The low-level details are the risk — a hard algorithm, an unfamiliar API |
| **Sandwich** | Both ends toward the middle | Some of each | Realistic default on most real systems |
| **Risk-oriented** | Hardest and riskiest parts first, regardless of level | Both | The main unknown could invalidate the design |
| **Feature-oriented** | One complete vertical feature at a time | Some of each | Deliverable increments matter; fits agile delivery |

**Choose deliberately and say which you chose and why.** The failure mode is drifting into
one by accident and discovering the interfaces do not meet.

In practice, prefer risk-oriented or feature-oriented for real systems: both surface the
expensive surprises early, and feature-oriented has the property that something demonstrable
exists at every point.

## Stubs and drivers

- A stub returns fixed, plausible values so a caller can be exercised before the real
  implementation exists.
- A driver calls a component so it can be exercised before its caller exists.
- Keep both honest: a stub that silently returns success hides a failure path. Make stubs
  able to return errors too, and make it obvious in logs that a stub is in use.
- Track stubs deliberately. A stub that reaches production is a defect with a long fuse —
  mark each one so it cannot be forgotten.

## The daily build and smoke test

The system should build and run a smoke test continuously — every commit in a modern setup.

- **The build must not stay broken.** A broken build blocks everyone, so fixing it outranks
  feature work.
- **The smoke test exercises the system end to end**, shallowly: does it start, serve a
  request, connect to its dependencies, complete one representative path? It is not a
  substitute for unit tests; it answers "is this fundamentally alive?"
- **Grow the smoke test as the system grows.** One that still tests only what existed in
  week one stops being evidence.
- **When you add a module, extend the pipeline in the same change** — build, test, lint.
  A module outside CI is untested by default.

Check whether CI configuration exists before assuming. If the project has none and it would
help, offer it; do not silently introduce pipeline infrastructure that the team has not
asked for.

## Interfaces between modules

- Agree the interface before building either side; it is the contract the integration rests on.
  *Where* the boundary should fall, and which way its dependency points, is a design question —
  see `architecture` before assembling.
- Each module barricades its own inputs. Data validated inside module A is untrusted at
  module B's boundary, even when the same person wrote both.
- Keep the dependency graph acyclic. A cycle between modules means they are one module
  pretending to be two.
- Version and document any interface crossing a deployment boundary. Separately deployed
  units are separately versioned whether or not you planned for that.

## Scaling formality to size

The amount of process that helps is a function of project size, and mismatching it is costly
in both directions.

- **Small (one or few developers, short-lived):** lightweight. Working code and tests over
  documents. Excess ceremony is the main risk.
- **Medium:** written interface contracts, a documented integration order, CI, code review.
- **Large (many developers, long-lived, high cost of failure):** explicit architecture,
  formal interface ownership, staged integration, and inspection rather than informal review.

Communication paths grow roughly as the square of team size, so what a large project buys
with formality is bounded communication — not bureaucracy for its own sake. Match the
project in front of you rather than defaulting to either extreme, and when a project grows
past its process, say so.
