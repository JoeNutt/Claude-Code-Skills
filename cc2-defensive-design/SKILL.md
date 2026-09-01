---
name: cc2-defensive-design
description: This skill should be used when validating external or untrusted input, designing where a system trusts its own data, writing assertions, adding error handling, or defining the boundary between untrusted and trusted layers. It covers the barricade pattern, assertions versus error handling, and choosing between correctness and robustness. Trigger phrases include "add validation", "handle exceptions", "secure this input", "sanitize this", "add error handling", "make this robust", "add assertions", "defensive checks", "what if this is null".
---

# Defensive Design

Assume invalid data will arrive. The goal is not to check everything everywhere — that is
how validation logic metastasizes through a codebase — but to check it **once, at a known
boundary**, so everything inside can be written against data it may trust.

## The barricade

Divide the software into two zones:

- **Outside the barricade** — data from users, network calls, files, environment variables,
  databases, other services, other teams' modules. All of it is hostile until proven otherwise.
- **Inside the barricade** — your own validated domain. Data here has already been checked.

The barricade itself is a thin layer of boundary routines or types whose entire job is to
convert untrusted input into trusted, well-formed domain data, and to reject everything else.

```
   untrusted input  ──▶  [ barricade: validate, coerce, reject ]  ──▶  trusted core
   (strings, JSON,          returns domain types or fails            (assumes valid,
    env, user, HTTP)                                                  no re-checking)
```

**Rules that follow from this:**

1. Validate at the boundary, completely, once.
2. Convert at the boundary too. Turn the string into the date, the integer, the enum, the
   validated ID — so that being *inside* is visible in the type, not just in a convention.
3. Do **not** re-validate inside. Repeated null checks and re-parsing deep in the core are
   an anti-pattern: they add noise, hide where the real check lives, and let two layers
   disagree about the rules.
4. Inside the core, a violated expectation is a **programmer error**, not bad input — so it
   is an assertion, not an error path.
5. When you cannot tell whether a routine is inside or outside, say so explicitly in its
   contract rather than defending both ways.

## Assertions versus error handling

The distinction is not about severity. It is about *whose fault it is*.

| | Assertion | Error handling |
|---|---|---|
| Handles | Conditions that must never occur | Conditions expected in production |
| Cause | A bug in our code | The world being the world |
| Examples | Invariant broken, impossible state, null where the type forbids it | Bad user input, network timeout, missing file, malformed payload |
| Behavior | Fail loudly and immediately | Degrade deliberately per policy |
| In production | May be compiled out; must never carry side effects | Always active |

Two consequences worth stating: never put logic with side effects inside an assertion, since
it may not run in a release build; and never use an assertion for something a user can
trigger — that is an error path with a message, not a crash.

## Choosing the failure response

Decide deliberately, and note *why* where it is not obvious:

- **Correctness** — never return a wrong answer; fail instead. Default for money, medical,
  safety, auth, anything persisted.
- **Robustness** — keep running with a degraded but safe result. Default for rendering,
  telemetry, best-effort background work.

The two genuinely conflict, so pick one per subsystem rather than mixing them arbitrarily.
Whichever you pick: fail fast and loudly at the point of detection. Never swallow an
exception into an empty handler, and never let a failure return a plausible-looking default
that propagates silently as if it were real data.

## Procedure

1. Locate the boundary. Name where untrusted data enters.
2. Ask whether a barricade already exists — if so, extend it rather than adding a second one.
3. At the boundary: validate everything, convert to domain types, reject with a specific,
   actionable message that never echoes secrets back.
4. Inside: strip redundant defensive checks that the barricade now guarantees, and replace
   genuine invariant checks with assertions.
5. Record the choice of correctness vs robustness for that subsystem.
6. Test the failure paths, not just the happy path — malformed, missing, boundary, hostile.

Detailed patterns — barricade placement in layered systems, error-return style trade-offs,
what to log versus what to return, and the common anti-patterns — are in
`references/barricade-patterns.md`. Read it when designing a new boundary or when the
existing validation is scattered and needs consolidating.
