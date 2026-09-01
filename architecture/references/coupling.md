# Coupling, Cohesion, and Module Boundaries

## Coupling is the root concept

Coupling is what most directly limits how reliably and sustainably software can be changed
and delivered. It is worth being precise about the relationship: **modularity, cohesion,
abstraction and separation of concerns are not four peers sitting alongside coupling — they
are the techniques by which coupling is managed.** That is the reason they matter.

So when weighing any of them, the question to ask is not "is this modular?" but "does this
reduce, or better place, the coupling — and at what cost?"

## Coupling cannot be removed, only placed

Components must communicate, so coupling is inherent. The engineering decision is **where it
sits, what form it takes, and which direction it points.**

Three questions that settle most cases:

1. **Which way does it point?** Toward the more stable thing. Volatile code may depend on
   stable code; never the reverse. A domain rule depending on a web framework is backwards —
   the framework changes far more often than the rule.
2. **How visible is it?** Coupling through a declared interface is manageable. Coupling
   through a shared global, an implicit call order, or an assumption about another module's
   internals is not, because nothing tells you when it breaks.
3. **How much must a caller know?** If using a module correctly requires knowing how it works
   inside, the abstraction has failed regardless of the interface.

### Forms of coupling, worst first

| Form | Shape | Why it hurts |
|---|---|---|
| **Semantic / behavioral** | Depending on another module's *behavior* rather than contract: call ordering, a side effect, "it returns sorted" | Compiles fine, breaks silently, invisible in review |
| **Shared mutable state** | A global, a shared cache, a singleton | No owner, no discipline, action at a distance |
| **Temporal** | A must run before B, unenforced | Works until someone reorders |
| **Structural / content** | Reaching into internals, `a.getB().getC()` | Binds you to a structure two modules away |
| **Control** | A flag telling the callee which branch to take | The caller is writing the callee's logic |
| **Data** | Passing the values needed, via a signature | The acceptable kind |

Aim to convert coupling downward through that table. Semantic coupling in particular should
be made explicit — if B genuinely requires A to have run, express it in the type or the
signature rather than in a comment.

### A second lens: coupling by consequence

The taxonomy above classifies coupling by its *form*. A complementary model (Nygard's)
classifies it by *what it stops you doing*, which is often the more actionable question at
system scale:

- **Developmental** — you can't release your change until I've finished mine.
- **Operational** — my service can't start unless yours is already running.

Both are design choices, not facts of life, and both are invisible in a form-based analysis:
two services can be beautifully decoupled at the code level and still be developmentally
coupled by a shared release train. Ask of any boundary: *can these be built, tested,
released, and run independently?* Where the answer is no, name which kind of coupling is
stopping it.

### Afferent and efferent

- **Efferent** — what this module depends on. High means fragile: many things can break it.
- **Afferent** — what depends on this module. High means rigid: changing it breaks many things.

A module with both high is the worst case, and is usually the `common`/`shared`/`utils`
bucket. Those modules acquire dependencies in every direction and become an undirected
coupling graph running underneath your intended architecture. Split them by concern.

**Stable things should be abstract; volatile things should be concrete.** A module that
everything depends on had better be either stable or an interface.

## Decoupling has costs

Loose coupling is a preference, not an absolute, and treating it as an absolute is itself a
design error.

**Decoupling usually means more code.** Introducing an interface, an adapter, a translation
layer or a data structure at a boundary adds code that does no business work. That is a real
cost, paid for real benefit. The common mistake is the reflex that "less code is good, more
code is bad" — at a boundary, the extra code is frequently the right trade.

**Coupling can be too loose.** Abstraction and indirection pushed past the point of value
produce systems that are hard to follow, hard to change as a whole, and sometimes materially
slower — indirection is not free at runtime. "We followed best practice" is not a defence for
a design nobody can trace.

**DRY is too simplistic as stated.** A single canonical representation of each behavior is
good advice *within* a function, a module, or a service — and reasonably up to the scope of
one repository or deployment pipeline. Applied *across* independently developed services or
modules it inverts: enforcing one shared representation couples them together, and that cost
usually exceeds the cost of the duplication. Two services each with their own notion of
"customer", translated at the boundary, are often better designed than two services sharing
one model.

The rule of thumb: **deduplicate within a boundary; translate across one.**

## Cohesion

Cohesion is the counterpart, and the two trade off: pursuing low coupling by splitting
aggressively produces modules with no coherent purpose, which is its own cost.

The practical test: **do the things in this module change together, for the same reason and at
the same time?** If half a module changes on every release and the other half hasn't changed
in two years, it is two modules.

The strongest form is functional cohesion — everything present contributes to one
well-defined job. The weakest is coincidental — things grouped because they are the same
*kind* of thing (all the controllers, all the helpers) rather than because they serve the
same purpose. That is exactly what layer-first folder structures produce.

## Where to draw a boundary

Good boundaries fall where:

- **Change rates differ.** Separate what changes weekly from what changes yearly.
- **The reason to change differs.** Different stakeholders, different domains.
- **A dependency should be inverted** — anything volatile the domain must not know about.
- **A team boundary exists.** Ownership boundaries and module boundaries should coincide.
- **The vocabulary changes.** Where "order" starts meaning something different, you are in a
  different context.

Bad boundaries fall where:

- Technical type differs but purpose does not.
- The split is speculative — "we might need this separately one day."
- Two sides must always be changed together anyway. That is one module with extra ceremony.

## Module boundary vs process boundary

**Do not use a network call to solve a design problem.** Splitting a tangled module into two
services does not decouple it; it converts a compile-time error into a runtime failure and
adds latency, partial failure, versioning, and distributed debugging.

Get the module boundary right *in the monolith* first. A clean internal boundary can be
extracted later; a bad one becomes a distributed bad one. Microservices are not the only
route to modularity, and treating them as the definition of it skips the actual work — which
is taking the boundaries and the protocols across them seriously.

Note the specific cost that independent deployability buys: services that deploy
independently **are not tested together**. That is the trade — release autonomy in exchange
for losing the guarantee that the combination works. Worth making deliberately.

Split into a separate process only for a reason that is genuinely about deployment or
runtime:

- Independent deployment cadence, and it demonstrably matters.
- Genuinely different scaling profile.
- Isolation of failure or of a security/trust boundary.
- Different technology genuinely required.
- Team autonomy at a scale where coordination cost is real.

"It's cleaner" is not one of these. Neither is "it's the modern approach."

## Testability and deployability as design feedback

Two properties worth optimizing for directly, because they act as continuous, honest feedback
on the design rather than as separate activities:

**Testability.** If a test is hard to write, the design is poor. Not "the tooling is
inadequate" — the design. The specific signals:

| Symptom | What it says |
|---|---|
| Heavy mocking needed | Too many collaborators, or the wrong ones |
| Needs a database, clock, or network to test a rule | Infrastructure reached the domain; a port is missing |
| Setup longer than the assertion | The unit does too much |
| Test asserts on internals | Behavior isn't exposed at the right level |
| Can't test without starting the whole app | No seams; everything is wired concretely |

Design for testability and you get modularity, cohesion, separation of concerns and loose
coupling as a consequence — they are the same properties viewed from a different angle.

**Deployability.** If releasing is slow, risky, or manual, feedback is slow, and slow feedback
degrades every decision that follows. Small, independently deployable, reversible changes are
an architectural property, not just a pipeline one — a design that forces coordinated releases
across five modules has made a structural choice.

## Working iteratively

Architecture is not decided once, up front. Take small steps, get feedback, and correct:

- Prefer the decision you can reverse. Where a choice can be deferred behind a port, defer it.
- Treat a structural change as an experiment with a stated expectation — then check it.
- Judge by evidence: is change actually getting easier, are tests getting simpler, is the
  dependency graph getting cleaner? "It follows the pattern" is not evidence.
- Refactor toward the boundary incrementally rather than pausing for a rewrite. See `tdd`.

A design that cannot be changed incrementally has already failed the property it most needed.
