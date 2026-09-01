# Threat Modeling — working guide

## Finding trust boundaries

A trust boundary is anywhere data or control crosses from a place with less trust to a place
with more. Work through this list against the system; each item is a boundary people
routinely fail to draw.

- **Client to server.** Everything client-side is attacker-controlled: form fields, hidden
  inputs, headers, cookies, JS validation, mobile app logic, the price in the cart payload.
  Client-side validation is a usability feature. It is never a security control.
- **Service to service**, including two services you wrote. They are deployed separately,
  versioned separately, and can be reached separately.
- **Your code to your datastore**, when anything else writes to it — another service, a
  migration, an admin, an older version of your own code.
- **Request handler to background job.** The queue is a boundary; the job runs later, with
  different privileges, on data the enqueuer controlled.
- **Tenant to tenant** in any shared system. The most common serious multi-tenant bug is a
  query missing its tenant predicate.
- **CI/CD to production.** The pipeline can deploy code, so it is at least as privileged as
  production. Anyone who can change the build can change what runs.
- **Third-party code to your process.** A dependency runs with your full privileges.
- **Untrusted file or payload to parser.** Uploads, imports, webhooks, and anything
  deserialized.
- **Privilege transitions** inside one process: before and after authentication, before and
  after a role check, admin vs user code paths.

## A worked STRIDE pass

Take one flow — "user uploads a profile image" — and walk it.

| Element | STRIDE | Threat | Mitigation |
|---|---|---|---|
| Upload endpoint | S | Request forged from another origin | Anti-CSRF token or SameSite cookies; verify Origin |
| Upload endpoint | E | Any user overwrites another's image | Authorize the object, not just the session — check ownership server-side |
| File content | T | File is a script, not an image | Verify content by parsing, not by extension or Content-Type; re-encode the image |
| File content | D | Decompression bomb exhausts memory | Cap dimensions and bytes *before* decoding; enforce a timeout |
| Storage path | E | Filename `../../etc/passwd` escapes the directory | Never use client filenames; generate your own; canonicalize then verify the prefix |
| Served image | I | Direct URL guessing exposes others' uploads | Authorize on read, not just on write; use unguessable ids as defence in depth, not as the control |
| Served image | E | SVG or HTML served inline executes script | Serve from a separate origin with `Content-Disposition: attachment` and a fixed Content-Type |
| Processing | D | Image library CPU exhaustion | Bound the work: size limits, timeouts, a separate pool |
| Audit | R | No record of who uploaded what | Log actor, object, action, outcome |

Two things this illustrates. One flow yields a dozen real threats, and most are authorization
and resource-limit failures rather than exotic attacks. And the mitigations are ordinary —
threat modeling's value is in *finding* them, not in inventing clever defences.

## Planning for failure

Controls fail. Assume a breach will happen and decide the response before you need it —
during an incident is the worst possible time to be designing one.

Answer these while the system is calm:

- **What happens if this is compromised?** Which data is exposed, which other systems become
  reachable from here, what can the attacker do next?
- **How would we know?** If nothing logs or alerts on it, the answer is "when someone tells
  us", and the breach runs for as long as it likes. See `sec-review-testing` on detection.
- **How do we revoke?** Every credential, token, key, and session needs a path to invalidate
  it quickly. A secret with no rotation path is a permanent compromise once leaked.
- **How do we contain?** Can the component be isolated or disabled without taking everything
  down? Least privilege pays off here: it bounds the blast radius in advance.
- **How do we recover?** Backups that have been restored at least once, a known-good state to
  return to, and a way to rebuild rather than clean.
- **What do we tell people, and who decides?** Disclosure obligations and the person
  authorized to act, agreed beforehand.

Design consequences worth taking now: keep the audit trail somewhere the compromised
component cannot rewrite, prefer short-lived credentials so exposure expires on its own, and
make rotation routine rather than an emergency procedure. A rotation path that has never been
exercised will not work the day it matters.

## The book's process, and the four questions

The source material runs threat modeling as: assemble the team, decompose the application
into a data flow diagram, determine the threats (STRIDE), rank them by decreasing risk
(DREAD), choose how to respond, then choose mitigation techniques.

The four questions in `SKILL.md` are the later, lighter restatement of the same loop, and
they are easier to apply to a single feature. Use whichever fits: the formal decomposition
for a system or a major subsystem, the four questions for a change. What must not be skipped
in either is the decomposition — threats are found per element and per flow, and a model
without a data flow yields generic worries rather than findings.

## Rating and prioritizing

Rate by **impact if exploited × how reachable it is**, and be concrete:

- Who can reach this — anonymous internet, any authenticated user, another tenant, an admin?
- What do they get — one record, all records, code execution, persistence?
- What does it cost them — a crafted request, or a stolen laptop and physical access?

Anonymous plus total loss is a stop-the-line finding. Authenticated-admin plus minor
disclosure usually is not.

**On DREAD:** the book teaches DREAD (Damage, Reproducibility, Exploitability, Affected
users, Discoverability) for rating. Microsoft itself abandoned it — the scores are
subjective, different reviewers produce very different numbers for the same bug, and the
arithmetic gives false precision. Use it as a prompt for *what to consider* if you like, but
prioritize with the reachability-and-impact questions above, or with an organizational
standard like CVSS where one is already in use.

## Secure defaults and deployment

The shipped configuration is the one most installations run forever.

- Features **off** by default; the user opts in to risk.
- **No default credentials.** Not "admin/admin", not a well-known key, not a seeded account.
  Generate at install or force creation on first run.
- **Deny by default** in every access rule; grant explicitly.
- **TLS on by default**, with certificate validation on. Never ship a "verify=false" default
  or leave a plaintext fallback available.
- **Least privilege at install:** no root/administrator unless genuinely required, minimum
  filesystem permissions, dedicated service account, minimum network exposure.
- **Bind to localhost by default**, not to all interfaces.
- Don't install what isn't needed — sample apps, debug endpoints, admin consoles, docs
  servers, default routes. Every one is attack surface someone forgot.
- **Debug and verbose errors off** in production builds, and no way to enable them remotely.
- Ship a way to **update**. Unpatchable software is a permanent vulnerability.
- Fail closed on misconfiguration: refuse to start with a missing secret rather than
  starting with a blank or default one.

## Privacy in the model

Privacy failures and security failures share most of their causes, and personal data raises
the impact of every other finding.

- **Minimize.** Data you do not collect cannot leak. Challenge each field's necessity.
- **Know where it goes.** Personal data crossing a trust boundary — to a log, an analytics
  provider, an error tracker, a support tool — is a disclosure path that needs modeling.
- **Retention is a control.** Data kept forever will eventually be breached; delete on a
  schedule and make deletion real, including in backups and derived stores.
- **Keep it out of logs, URLs, and error messages** — the three places it most often ends up
  by accident.
- Support access and deletion requests by design; retrofitting them is far harder.

## When to re-model

Threat models go stale. Revisit when a trust boundary moves, a new external interface is
added, authentication or authorization changes, a component's privileges change, a new data
class is handled, or a dependency with broad access is added. A model that hasn't changed
while the system has is describing a system you no longer run.
