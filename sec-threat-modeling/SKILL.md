---
name: sec-threat-modeling
description: This skill should be used when starting a new feature, service, or system, when changing how components trust each other, when reviewing an architecture for security, or when deciding what could go wrong before code is written. It covers STRIDE threat modeling, trust boundaries, attack surface reduction, secure defaults, and the core security principles. Trigger phrases include "threat model", "is this design secure", "what could go wrong", "security review of this architecture", "new service", "new endpoint", "attack surface", "trust boundary", "secure by default", "am I over-engineering the security".
---

# Threat Modeling and Security Principles

Security is a design property. Most serious vulnerabilities are decisions, not typos — and
a decision is far cheaper to change before it is implemented. Model the threats first.

## The principles

These decide most questions before you reach for a technique.

- **Assume external systems are insecure.** Every byte from outside your trust boundary is
  attacker-controlled until validated — and this extends past user input to any system you
  don't control: a partner API, a shared database, a dependency, another team's service.
  See `sec-input-validation`.
- **Minimize the attack surface.** Every endpoint, parameter, port, file format, permission,
  and dependency is a way in. The most reliable way to secure something is to not expose it.
- **Least privilege.** Every component, process, credential, and token gets the minimum
  rights needed, for the minimum time. See `sec-authz-least-privilege`.
- **Defense in depth.** Assume any single control fails. Ask what the next one is; if there
  isn't one, that control is load-bearing and needs to be excellent.
- **Employ secure defaults.** The out-of-the-box configuration is the one most people run.
  Ship with the restrictive setting, features off, no default credentials, TLS on.
- **Fail to a secure mode.** An error must leave the system in a safe state.
  `if (isAuthorized())` that throws must deny, never proceed. Never let an exception path
  skip a check.
- **Plan on failure.** Controls get breached and bugs ship — assume it will happen to you.
  Know in advance what you do when the service is compromised, a credential leaks, or data
  is exfiltrated. "It'll never happen" is not a plan, and an incident is the worst possible
  time to design the response.
- **Security features != secure features.** Adding SSL, or encryption, or an auth library
  does not make software secure. The question is whether the *correct* control is applied to
  the thing that actually needs defending — encrypting a channel nobody attacks while the
  authorization check is missing is effort spent for nothing. Threat modeling is how you
  find out which is which.
- **Don't mix code and data.** Where an interpreter cannot distinguish your structure from
  an attacker's content, you have an injection vulnerability. Separate them structurally.
  See `sec-injection-defense`.
- **Never depend on obscurity alone.** Assume the attacker has the source, the binary, and
  time. Obscurity is acceptable as an extra layer, never as the control.
- **Backward compatibility will always give you grief.** An insecure protocol, format, or
  default that shipped is one you will be asked to support forever, because clients will not
  all upgrade. Weigh that when choosing one — and where a compatibility fallback exists, it
  *is* the security level, since an attacker will simply request it.
- **Fix security issues correctly.** When you find one, fix the root cause rather than the
  symptom, then **go looking for the same mistake elsewhere** — whoever wrote it likely made
  it more than once. Add a regression test, and ask what would prevent the whole class.
- **Learn from mistakes.** Making a mistake is human; making the same one repeatedly is a
  process failure. Capture what allowed it — a missing check, an unsafe default, an API easy
  to misuse — and change that, not just the line.
- **Separation of duties.** No single component or credential should be able to complete a
  sensitive action end to end without a second check.
- **Don't invent your own crypto or auth protocol.** See `sec-crypto-secrets`.

## Modeling the threats

Four questions, in order:

### 1. What are you building?

Sketch the data flow: the processes, the data stores, the external entities, and the flows
between them. Then draw the **trust boundaries** — every place data crosses from less
trusted to more trusted.

Boundaries people miss: between your service and a service you also wrote; between the
browser and your API (everything client-side is attacker-controlled); between your code and
your own database when anything else can write to it; between a request handler and a
background job; between tenants in a shared system; between CI and production.

**Every arrow crossing a boundary is where a threat lives.** If nothing in your model
crosses a boundary, the model is wrong.

### 2. What can go wrong?

Walk each element against **STRIDE**:

| | Threat | Property violated | Ask |
|---|---|---|---|
| **S** | Spoofing | Authentication | Can someone claim to be another user, service, or process? |
| **T** | Tampering | Integrity | Can data be modified in transit, at rest, or in the client? |
| **R** | Repudiation | Non-repudiation | Can someone deny doing it? Is there a trustworthy log? |
| **I** | Information disclosure | Confidentiality | Can someone read what they shouldn't — including via errors, timing, or metadata? |
| **D** | Denial of service | Availability | Can someone exhaust CPU, memory, storage, connections, or quota? |
| **E** | Elevation of privilege | Authorization | Can someone do something they aren't entitled to? |

Apply it per element, not to the system as a whole — "can this endpoint be spoofed?" yields
findings, "is the system secure?" does not.

### 3. What are you going to do about it?

For each threat, choose deliberately: **mitigate** (add a control), **eliminate** (remove the
feature or the exposure), **transfer** (make it another component's responsibility,
explicitly), or **accept** (with the reason written down). Accepting a risk is legitimate.
Accepting it silently is not.

### 4. Did you do a good job?

Every threat needs a control, an owner, or a recorded acceptance — and every control needs a
test. A mitigation nobody verified is a belief.

## Where the modern risks are

The current OWASP Top 10 (2025) is a useful checklist against a finished model. In rank
order: broken access control; security misconfiguration; **software supply chain failures**;
cryptographic failures; injection; insecure design; authentication failures; software and
data integrity failures; security logging and alerting failures; and **mishandling of
exceptional conditions**.

Two of those are worth flagging because they are newly ranked and easy to leave out of a
model: your dependencies and build pipeline are part of your attack surface, and error paths
are a category of vulnerability in their own right, not just a robustness concern.

## Proportionality

Match the effort to what is actually at risk. A personal CLI tool that reads local files does
not need a formal threat model; a multi-tenant service handling payments does. Ask what an
attacker gains, what they'd have to do, and what it costs if they succeed.

Say so plainly when a security measure isn't warranted. Recommending controls nobody needs
trains people to ignore the recommendations that matter.

Deeper material — a worked STRIDE pass, boundary-identification heuristics, secure-defaults
and deployment checklists, and why DREAD is no longer recommended for rating — is in
`references/modeling-guide.md`.
