# Barricade Patterns — placement, style, and anti-patterns

## Where the barricade goes in a layered system

The barricade belongs at the outermost layer that still understands the domain — close
enough to the edge that nothing untrusted slips past, far enough in that it can produce
real domain types.

```
  HTTP handler / CLI parser / queue consumer / file reader
        │   raw strings, bytes, JSON, form data
        ▼
  ┌─────────────────────────────────────────────┐
  │  BARRICADE                                  │
  │  parse → validate → convert → reject        │
  │  output: domain types, guaranteed well-formed│
  └─────────────────────────────────────────────┘
        │   OrderId, EmailAddress, Money, UtcInstant
        ▼
  services / domain logic / persistence
        (assume valid; assert invariants only)
```

Typical barricade sites:

- Request handlers and controllers — before anything reaches a service.
- Deserialization of any external payload.
- Configuration and environment loading, at startup, so a bad config fails immediately.
- Reads from a datastore your code does not exclusively own.
- Any public entry point of a library others call.

### Trust boundaries are per-system, not universal

Data trusted inside service A is untrusted at service B's edge, even if you wrote both.
Each deployable unit barricades its own inputs. "It was validated upstream" is not a
guarantee you can hold when upstream is deployed separately and versioned independently.

Your own database is a trust boundary too when other processes, migrations, or humans can
write to it. Data written by an older version of your own code is external data.

## Making trust visible in types

The strongest form of the barricade encodes validation in the type system, so the compiler
enforces what a comment would otherwise merely claim.

```
# Weak: every consumer must wonder whether this was checked
function createUser(email: string, age: int)

# Strong: unconstructable without passing validation
function createUser(email: EmailAddress, age: AdultAge)
```

Where the type constructor is the only way to build the value, and it validates, the
question "has this been checked?" stops being askable. In dynamically typed languages,
approximate it with a single named constructor or factory that every call site must use,
and keep the raw form out of the domain.

Parse, don't validate: prefer returning a *converted* value over returning a boolean about
an unconverted one. A function that answers "is this a valid date string?" leaves the caller
holding a string; one that returns a `Date` or fails leaves them holding a date.

## Error-return style

Language idiom governs; consistency within a codebase matters more than the choice itself.

| Style | Fits | Watch for |
|---|---|---|
| Exceptions | Exceptional, non-local failures | Catch-all handlers; swallowing; using them for ordinary control flow |
| Result / Either types | Expected, recoverable failures | Unchecked ignoring of the error arm |
| Error return values | C-style and Go-style codebases | Silently unchecked returns |
| Sentinel / null return | Rarely a good default | Indistinguishable from a legitimate value |

Whatever the style: an error must carry enough context to diagnose the failure, and must
not be discardable by accident.

## What to log versus what to return

- **Return** to the caller: what they can act on. Which field, what was wrong, what is
  acceptable. Never internal structure, stack traces, SQL, or secrets.
- **Log** internally: the full diagnostic context — correlation id, the offending value if
  it is not sensitive, the code path.
- **Never** put credentials, tokens, keys, full card numbers, or personal data in either.
  Redact at the point of formatting, not at the point of reading.

An error message the user cannot act on is noise; an error log the operator cannot correlate
is worse than nothing.

## Anti-patterns

**Defensive checking everywhere.** Null checks at every level, the same field re-validated
in four routines. Symptom of an absent or untrusted barricade. Fix the boundary, then
delete the interior checks.

**The silent default.** `catch { return [] }`, `catch { return 0 }`. A failure becomes
plausible-looking data and surfaces later, far from its cause, as a wrong answer rather
than an error.

**The empty catch.** Discards the only evidence of the fault. If a failure is genuinely
ignorable, say so in a comment explaining why — that comment is the whole justification.

**Assertion as validation.** Asserting on user input crashes the process for a condition a
user can trigger at will, and vanishes entirely in builds where assertions are disabled.

**Assertions with side effects.** `assert(deleteExpired() > 0)` — the work disappears in
release builds. Assertions read state; they never change it.

**Validation duplicated in two layers with different rules.** Two barricades that disagree
are worse than one, because the stricter one silently defines the real contract and the
looser one lies about it.

**Barricade too deep.** If untrusted data reaches business logic before validation, the
barricade is misplaced — every routine it passed through was implicitly a boundary routine
without being written as one.

## Reviewing an existing boundary

1. Where does untrusted data enter? List every entry point, not just the obvious one.
2. Is there exactly one validation site per entry point, or several disagreeing ones?
3. Does the boundary output domain types, or does it pass raw strings inward?
4. Which interior checks are now redundant and can be deleted?
5. Which interior checks are genuine invariants and should become assertions?
6. Does every failure path have a test — malformed, missing, boundary, and hostile input?
