---
name: architecture
description: This skill should be used when deciding how a system is structured rather than how a single routine is written - where a piece of logic belongs, how layers or modules are separated, which direction dependencies point, how the domain is isolated from frameworks and infrastructure, or whether something should be a separate service. Trigger phrases include "where should this go", "how should I structure this", "what layer", "architecture", "should this be a service", "dependency direction", "ports and adapters", "hexagonal", "clean architecture", "domain model", "coupling", "should I split this", "this is hard to test".
---

# Architecture

Architecture is the set of decisions that are expensive to reverse: what depends on what, and
where the boundaries sit. Everything else — how a routine is written, how a class is shaped —
is construction, covered by `cc2-construction`.

The purpose is not layering for its own sake. It is to **keep the decisions that matter
independent of the decisions that don't**, so business rules survive a change of database,
framework, or delivery mechanism, and so the important parts can be tested without any of them.

## The one rule

**Source code dependencies point only inward, toward higher-level policy.**

Nothing in an inner circle may name anything in an outer circle — no class, function, or
variable. The business rules must not know that a web framework, an ORM, or a message queue
exists.

```
        ┌─────────────────────────────────────────┐
        │  Frameworks & Drivers                   │   web, DB, UI, queue,
        │   ┌─────────────────────────────────┐   │   filesystem, external APIs
        │   │  Interface Adapters             │   │   controllers, presenters,
        │   │   ┌─────────────────────────┐   │   │   repositories, mappers
        │   │   │  Use Cases              │   │   │   application-specific rules,
        │   │   │   ┌─────────────────┐   │   │   │   orchestration
        │   │   │   │  Entities       │   │   │   │   enterprise rules,
        │   │   │   └─────────────────┘   │   │   │   least likely to change
        │   │   └─────────────────────────┘   │   │
        │   └─────────────────────────────────┘   │
        └─────────────────────────────────────────┘
                  dependencies point ►►► inward
```

The circles are a guide, not a quota. Some systems need three, some five. What is
non-negotiable is the **direction**.

## Ports and adapters — the same rule, more usable terms

Clean Architecture is a synthesis of earlier work: Cockburn's **ports and adapters**
(hexagonal, 2005) and Palermo's onion (2008). They are not competing choices, and it is worth
knowing that the vocabulary differs while the rule does not.

- A **port** is an interface *owned by the inside* — the domain declares what it needs
  (`OrderRepository`, `PaymentGateway`) or what it offers.
- An **adapter** lives outside and implements that port for a specific technology — Postgres,
  Stripe, an HTTP handler.
- **Driving** adapters (left) call in: HTTP, CLI, tests, message consumers.
- **Driven** adapters (right) are called by the domain: databases, external services, clocks.

In practice, prefer this vocabulary. "Where does this go?" is much easier to answer as "is
this a port, an adapter, or domain?" than by counting rings — and it makes the crucial point
explicit: **the interface belongs to the inner side, not the implementer.**

## Crossing a boundary

Control flows outward but dependencies must point inward, so invert them:

1. The inner layer **declares an interface** for what it needs.
2. The outer layer **implements** it.
3. Something at the edge — composition root, DI container, `main` — wires them together.

`main` is the outermost thing in the system: it knows every concrete type, and nothing knows
about it.

**Pass simple data across boundaries.** Not entities, not ORM rows, not framework request
objects. A plain structure shaped for the receiving side. The moment a database row reaches
a use case, the database has become a dependency of your business rules, whatever the folder
structure says.

## Managing complexity

The properties worth designing for. They are the same ideas `cc2-construction` applies to
routines, applied to modules:

- **Modularity** — you can work on one part without holding the rest in your head.
- **Cohesion** — things that change together live together. This is the deeper reason to
  organize by feature or domain concept rather than by technical type: `controllers/`,
  `services/`, `models/` scatters every real change across three directories.
- **Separation of concerns** — one module, one reason to change.
- **Information hiding and abstraction** — expose the least that works. Abstract "enough that
  you can change your mind later," accepting that all non-trivial abstractions leak.
- **Managing coupling** — you cannot eliminate coupling; components must communicate. You can
  choose where it sits and what form it takes. Push it to the edges, make it explicit, and
  keep the direction pointing at stability.

## Testability is the signal

The most reliable feedback available on a design: **if tests are easy to write, the design is
probably good; if they are hard, it is probably not.**

Take that literally. Needing heavy mocking, a database, a live clock, or network access to
test a business rule is not a testing problem to be solved with better tooling — it is the
design telling you a dependency points the wrong way. Fix the direction rather than reaching
for a more powerful mock.

This is where architecture and `tdd` meet: the domain sits behind ports, so it is tested with
trivial in-memory fakes and no infrastructure at all.

## Proportionality

Full layering on a small tool is waste, and the ceremony makes it *harder* to change, not
easier. Scale to what the system actually is:

- **Script / small tool:** no layers. Keep I/O separable from logic and stop.
- **Service with real business rules:** ports and adapters. Domain isolated, infrastructure
  behind interfaces.
- **Large or long-lived system:** explicit layers, enforced dependency direction, documented
  boundaries.

The test is whether the business rules are complex enough to be worth protecting. If the
system is a thin shell over CRUD, say so rather than building four layers to move a field
from a form to a table.

**Defer what you can.** A good architecture keeps options open — the decisions it postpones
are as important as the ones it makes. If the database or framework choice can be deferred
behind a port, defer it.

## Common failures

| Failure | Why it costs |
|---|---|
| Framework or ORM types in the domain | The rule is broken regardless of folder names |
| Entities returned straight to the web layer | Every schema change becomes an API change |
| Interface defined next to its implementation | The dependency still points outward; the port belongs inside |
| Layers by technical type (`controllers/`, `models/`) | Low cohesion — every change touches every folder |
| Anemic domain: data classes plus "service" procedures | Business rules live in the outer layers |
| Pass-through layers that only forward calls | Ceremony with no isolation |
| A "shared"/"common" module everything depends on | Becomes a second, undirected coupling graph |
| Cyclic dependencies between modules | They are one module pretending to be two |
| Splitting into services to fix a coupling problem | Distribution adds failure modes; it does not decouple |

`references/layering.md` covers what belongs in each layer and how to decide where a given
piece of code goes. `references/coupling.md` covers coupling types, module boundaries,
cohesion, and when a boundary should become a process boundary.
