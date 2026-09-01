---
name: sec-review-testing
description: This skill should be used when reviewing code or a change for security, writing security tests, deciding what to log for detection and audit, designing error handling that does not leak information, or checking dependencies and the build pipeline for supply chain risk. Trigger phrases include "security review", "review this for vulnerabilities", "is this secure", "security test", "fuzz", "pen test", "audit this code", "what should I log", "error message", "dependency vulnerability", "supply chain", "before we ship".
---

# Security Review, Testing, and Detection

Reviewing for security is not ordinary code review with more suspicion. It is a distinct
pass with a distinct question: **not "is this correct?" but "how would I attack this?"**

## Where to look, in priority order

Never review a large codebase uniformly — you'll run out of attention before you reach the
risky part. Rank by exposure:

1. **Code handling untrusted input** — request handlers, parsers, deserializers, file
   uploads, webhooks.
2. **Authentication and authorization** — every check, and every path that skips one.
3. **Anything crossing a trust boundary** — see `sec-threat-modeling`.
4. **Code running with elevated privilege**, or that grants it.
5. **Cryptography, secrets, and session handling.**
6. **Anything recently changed**, especially under time pressure.
7. Code with weak test coverage and no owner.

Then make multiple focused passes rather than one pass looking for everything at once. A
pass per question — injection, then authorization, then secrets — finds substantially more
than a single undirected read.

## The review questions

For each item above:

- **Where does the data come from, and who controls it?** Trace it to its source. "It comes
  from the database" only defers the question to whoever wrote the row.
- **What happens if it is hostile?** Too long, wrong type, negative, null, empty, unicode,
  encoded twice, someone else's id.
- **What is trusted here, and why?** Name the guarantee and find the code providing it.
- **What does this assume?** Assumptions are where the vulnerabilities are.
- **Is the check reachable from every path?** Including error paths, retries, background
  jobs, admin tools, and the second step of a wizard.
- **What happens when this fails?** Does it fail closed?

Cross-check the finished review against the OWASP Top 10 (2025): broken access control,
security misconfiguration, software supply chain failures, cryptographic failures, injection,
insecure design, authentication failures, software and data integrity failures, logging and
alerting failures, and mishandling of exceptional conditions.

## Security testing

Ordinary tests confirm intended behavior. Security tests confirm that unintended behavior is
**prevented**, which is a different exercise.

- **Test the denial**, not just the success. A test proving an owner can read proves nothing
  about a stranger. Every access rule needs a negative test, and multi-tenant systems need
  fixtures with at least two tenants.
- **Test with hostile input:** oversized, malformed, wrong type, boundary values, encoded and
  double-encoded, null bytes, unicode, traversal sequences, injection payloads.
- **Fuzz every parser handling untrusted input.** The highest-yield technique available for
  parsers, and it finds cases nobody would think to write.
- **Automate in CI:** dependency scanning, secret scanning, static analysis, and the
  sanitizers where the language needs them.
- **Regression-test every vulnerability you fix.** A security bug without a test is one
  refactor from returning.

No single technique catches most defects — review, testing, scanning, and fuzzing each catch
a partly different set. A clean result from one is not evidence of security.

## Supply chain

Your dependencies run with your privileges, and this is now among the most exploited risk
categories — it postdates most secure-coding material, so it is easy to leave out.

- Keep an inventory of what you depend on, transitively.
- Scan for known vulnerabilities continuously, not at release.
- **Pin versions and commit a lockfile.** Verify integrity hashes.
- Vet before adding: is it maintained, widely used, reasonably scoped? A tiny convenience
  package is rarely worth a new supply-chain entry.
- Watch for typosquats and sudden maintainer changes.
- **Treat the build pipeline as production infrastructure** — it can deploy code, so it is at
  least as privileged. Secrets in CI need the same care as secrets in production, and the
  pipeline needs its own least-privilege review.
- Verify artifact provenance where the ecosystem supports it.

## Error handling that doesn't leak

Error paths are a vulnerability class of their own, now ranked in the Top 10 in their own
right — mishandled exceptional conditions.

- **Fail closed.** An error in an authorization or validation path must deny. Never let an
  exception route around a check.
- **Two audiences, two messages.** The user gets an actionable, generic message; the log gets
  full diagnostic context. Never return stack traces, SQL, internal paths, versions, or
  configuration to a caller.
- **Don't let errors enumerate.** "Invalid username" versus "invalid password" tells an
  attacker which accounts exist; so does a different response time. Same for "not found"
  versus "forbidden" where existence is sensitive.
- **Never swallow an exception** in security-relevant code. A silent catch is a bypass with
  no evidence.
- Don't leave a partially-applied state behind on failure — an operation that half-succeeded
  is frequently exploitable.

## Logging for detection

Logging failures are ranked in the Top 10 because an undetected breach is an unbounded one.

**Log:** authentication attempts and outcomes, authorization **denials**, privilege changes,
password and MFA changes, access to sensitive data, administrative actions, input-validation
rejections, and configuration changes. Include who, what, when, where from, and the outcome.

**Never log:** passwords, tokens, keys, session ids, full card numbers, or personal data
beyond what is needed. Redact at the formatting point, and use structured fields so untrusted
values cannot forge entries.

Make logs **useful**: a correlation id through the request path, synchronized clocks,
tamper-resistant retention long enough to investigate. And alert on the signals that indicate
an attack in progress — repeated authorization denials, credential stuffing patterns,
unusual data volumes. A log nobody reads and nothing alerts on is storage, not detection.

## Reporting findings

For each: **location** (`file:line`), **what** the flaw is, **how it would be exploited** —
concretely, with the input — **impact** if exploited, and the **fix**.

Rank by exposure and impact. Anonymous remote code execution outranks a missing header,
however many headers are missing. Say clearly when you found nothing significant; padding a
report trains people to skim the next one. And be explicit about what this review could not
cover — logic you could not reach, behavior requiring runtime testing, dependencies not
examined.

A multi-pass method, a per-category checklist, and guidance on prioritizing are in
`references/review-method.md`.
