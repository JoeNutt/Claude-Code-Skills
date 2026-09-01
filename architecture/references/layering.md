# Layering — what goes where

## The decision procedure

When unsure where a piece of code belongs, ask in this order:

1. **Would this rule still exist if the product were a paper form?** → Entity / domain. A
   business truth independent of any software.
2. **Does it orchestrate a specific thing a user or system does?** → Use case. It coordinates
   domain objects and ports; it holds no business truth of its own.
3. **Does it translate between the domain and something outside?** → Adapter. Controllers,
   presenters, repository implementations, mappers, serializers.
4. **Is it a specific technology?** → Framework / driver. The outermost ring, wired in `main`.

If a piece of code answers "yes" to two of these, it is doing two jobs. Split it.

## What belongs in each layer

### Entities / domain
- Business rules and invariants that hold regardless of the application.
- Domain types that make illegal states unrepresentable — `Money`, `EmailAddress`, `OrderId`.
- Pure behavior: no I/O, no clock, no randomness, no framework imports.

A domain model with no behavior — only fields and getters — means the rules are somewhere
else, usually in "service" classes in an outer layer. That is the anemic domain, and it
gives up the whole benefit of having a domain layer.

### Use cases / application
- One class or function per thing the system does: `PlaceOrder`, `CancelSubscription`.
- Orchestration: fetch via ports, invoke domain behavior, persist via ports.
- Transaction boundaries and authorization checks usually sit here.
- **Declares the ports it needs.** The interfaces live here (or in the domain), never with
  their implementations.

Use cases contain *application*-specific rules — "an order may only be cancelled within 30
days" is a domain rule; "cancelling sends an email and refunds" is a use case.

### Interface adapters
- Controllers and handlers: translate a request into a use-case call. Thin.
- Presenters and view models: translate results into a response shape.
- Repository implementations: satisfy a domain-declared port using a real datastore.
- Mappers between persistence/wire models and domain types.

**Adapters are where the mapping tax gets paid**, and paying it is the point. The duplication
between a database row, a domain entity and an API response is not accidental duplication to
eliminate — they change for different reasons and each is the right shape for its side.

### Frameworks and drivers
- The web framework, ORM, message broker, cloud SDKs, filesystem.
- Configuration and `main` — the composition root that constructs concrete types and injects
  them.

Nothing depends on this layer except by interface.

## Organizing folders

Prefer **by feature, then by layer**:

```
orders/
  domain/          Order, OrderLine, pricing rules, OrderRepository (port)
  application/     PlaceOrder, CancelOrder
  adapters/
    http/          OrderController
    persistence/   SqlOrderRepository
billing/
  domain/ ...
```

over **by layer, then by feature**:

```
controllers/     OrderController, BillingController, UserController
services/        OrderService, BillingService, UserService
models/          Order, Invoice, User
```

The first keeps things that change together in one place. The second guarantees every change
touches three directories, and makes it invisible when `billing` starts reaching into
`orders`' internals.

Whatever the layout, **enforce the dependency direction with a tool** — a lint rule, an
architecture test, a module system, a compiler boundary. A convention documented in a README
and enforced by nothing will be violated, silently, and usually within weeks.

## Ports: naming and ownership

- Name a port for **what the domain needs**, not what implements it: `OrderRepository`, not
  `PostgresGateway`; `PaymentGateway`, not `StripeClient`.
- The port lives with the code that *uses* it. This is the part most often got wrong — an
  interface sitting in the same package as its single implementation has inverted nothing.
- One port per concern. A single `Database` interface with forty methods is not a boundary.
- **A port exposes only the subset you actually use**, not a mirror of the API behind it.
  Your code almost never needs every detail of a third-party service, so the port is nearly
  always a *simpler* thing than the API it fronts — expressed in your vocabulary, not the
  vendor's. A port that mirrors an SDK one-for-one has renamed the coupling, not removed it.
  Done properly, the code on the inside doesn't know the vendor exists.
- Ports for anything non-deterministic: clock, randomness, ids, filesystem, network. This is
  what makes the domain testable without infrastructure.

## Testing by layer

| Layer | Test with | Not with |
|---|---|---|
| Entities | Plain unit tests, no doubles | Any infrastructure |
| Use cases | In-memory fakes for ports | Mocks asserting call order |
| Adapters | Integration tests against the real technology | Mocks of the driver |
| Whole system | A few end-to-end tests through real entry points | Everything |

Prefer **fakes over mocks** for ports — a working in-memory implementation tests behavior,
where a mock asserting a call sequence tests your current implementation and blocks the
refactor step. See `tdd`.

If a use case needs elaborate mocking, the ports are wrong: too many, too fine-grained, or
leaking infrastructure concepts into their signatures.

## Common questions

**Where do validation rules go?** Structural validation (is this well-formed?) at the
boundary — see `sec-input-validation`. Business validation (is this allowed?) in the domain.
Both, and they are different jobs.

**Where does authorization go?** The decision belongs in the use case or domain, where the
actor and the object are both known. Enforcement may also sit at the edge as defence in
depth. See `sec-authz-least-privilege`.

**Where do transactions go?** The use case — it knows the unit of work. Not the repository,
which cannot see the whole operation.

**Where do domain events go?** Raised by the domain, dispatched by the use case, delivered by
an adapter.

**What about DTOs everywhere?** Between boundaries, yes. Within a layer, no — mapping for its
own sake is pass-through ceremony.
