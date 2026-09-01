# Security Review Method

## The multi-pass approach

One pass looking for everything finds less than several passes each looking for one thing.
Attention is the scarce resource; spend it deliberately.

**Pass 0 — Orient.** What does this component do, what does it trust, and where are its trust
boundaries? Identify entry points: routes, message handlers, jobs, CLI arguments, file
readers, webhook receivers. You cannot review what you have not enumerated.

**Pass 1 — Input.** Every entry point: what arrives, who controls it, what validates it,
before or after canonicalization? See `sec-input-validation`.

**Pass 2 — Authorization.** Every action: authenticated, authorized, and is the *object*
checked or only the action? Every read-by-id from user input is a candidate. See
`sec-authz-least-privilege`.

**Pass 3 — Interpreters.** Every query, command, template, path, redirect, outbound fetch,
deserializer. Is the defence structural or is it escaping? See `sec-injection-defense`.

**Pass 4 — Secrets and crypto.** Algorithms, key storage, randomness, secrets in the
repository or logs. See `sec-crypto-secrets`.

**Pass 5 — Error and failure paths.** Does everything fail closed? What leaks in messages?
Any swallowed exceptions in security-relevant code?

**Pass 6 — Configuration and dependencies.** Defaults, debug flags, exposed endpoints, CORS,
cookie flags, security headers, dependency vulnerabilities, CI secrets.

Record what you covered in each pass. An unfinished review that says where it stopped is
useful; one that implies completeness it doesn't have is worse than none.

## Prioritizing what to review

You will not review everything. Rank by exposure:

| Priority | Characteristic |
|---|---|
| Highest | Reachable anonymously from the internet; parses untrusted input; performs authn/authz |
| High | Runs with elevated privilege; handles secrets, money, or personal data |
| Medium | Reachable by authenticated users; crosses a trust boundary |
| Lower | Internal-only, no untrusted input, no privilege |

Then weight by change: recently modified code, code written under deadline, and code with no
tests and no owner.

## Per-category checklist

### Input
- Every entry point enumerated, including non-HTTP ones.
- Allowlist rather than denylist; reject rather than clean.
- Canonicalize → validate → use, in that order, decoding exactly once.
- Size, depth, count, and time limits enforced **before** expensive work.
- Parser hardening explicit: XML entities off, safe YAML loader, archive and image caps.
- No native deserialization of untrusted data.

### Authorization
- Deny by default; a new route without a policy fails closed.
- Object-level checks, not just role checks; prefer ownership-scoped queries.
- Tenant predicate enforced structurally, not per query.
- Re-checked after each workflow step and redirect.
- No client-asserted identity, role, tenant, or price.
- Policy-lookup errors deny.
- Negative tests exist.

### Injection
- Parameterized queries; identifiers allowlisted.
- No shell; argument arrays to direct exec.
- Contextual output encoding; auto-escaping on; every raw/unsafe call justified.
- Redirect targets and outbound fetch destinations allowlisted.
- Templates never built from user data.
- Structured logging; no untrusted data in format strings.

### Crypto and secrets
- No custom crypto; no MD5/SHA-1 for security; no ECB or unauthenticated CBC.
- Passwords: memory-hard, salted, parameters stored and current.
- CSPRNG for anything security-bearing; nonces unique per key.
- No secrets in source, history, images, or CI config.
- Rotation possible without downtime.
- Constant-time comparison for tokens and MACs.
- TLS verification on, everywhere, with no disable flag.

### Errors, logging, configuration
- Fail closed on every security-relevant error path.
- No stack traces, SQL, paths, or versions returned to callers.
- No enumeration through differing messages or timings.
- Security events logged; secrets and personal data not logged.
- Alerting exists on the signals that matter.
- Debug off, defaults restrictive, no default credentials, unnecessary endpoints removed.
- Cookie flags, security headers, CORS reviewed.

### Supply chain
- Lockfile committed; versions pinned; integrity hashes verified.
- Vulnerability scanning in CI, not at release.
- New dependencies vetted for maintenance and scope.
- Pipeline treated as production: least privilege, scoped secrets.

## Writing up a finding

```
[Severity] Short title
Location:  path/to/file.ext:LINE
Issue:     What is wrong, in one sentence.
Attack:    Concretely how it is exploited - the actual request or input.
Impact:    What the attacker gains. Whose data, how much, what access.
Fix:       The specific change. A snippet where the shape is not obvious.
```

The **Attack** line is what separates a finding from a worry. If you cannot describe the
input that exploits it, mark it as "needs verification" and say so rather than asserting a
vulnerability you have not demonstrated.

## Severity

| Level | Roughly |
|---|---|
| Critical | Unauthenticated RCE, full authentication bypass, mass data exposure |
| High | Authenticated privilege escalation, cross-tenant access, injection reaching data |
| Medium | Requires unusual conditions, limited scope, or meaningful attacker effort |
| Low | Defence in depth, hardening, information disclosure of little value |

Judge by **who can reach it** and **what they get**. A theoretical flaw behind three
authentication layers is not critical because the technique is impressive, and a missing
security header is not high because a scanner flagged it in red.

## Honesty

- Don't report what you haven't verified. Mark uncertainty as uncertainty.
- Don't claim a clean review means secure code. Say what you covered and what you didn't.
- Don't inflate severity to be heard, and don't pad the count — both cost you the reader's
  attention on the finding that mattered.
- Recommending controls disproportionate to the risk is its own failure: it trains people to
  ignore recommendations.
