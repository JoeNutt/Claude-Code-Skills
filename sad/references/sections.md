# Section formats

Templates for the sections that carry most of a SAD's weight. Use the formats exactly —
comparability across documents is the point of fixing them.

---

## 6. User stories

Governed by the `sad-user-stories` skill. Load it.

---

## 7. Functional requirements

```
| ID   | Requirement                                              | Serves |
|------|----------------------------------------------------------|--------|
| FR-1 | The system records an expense against a case, with an     | US-4   |
|      | amount, date, category and uploaded receipt.              |        |
| FR-2 | The system rejects an expense whose receipt is absent.    | US-4   |
```

Each requirement is a single testable statement of behaviour, in the present tense, with the
system as the subject. `Serves` names the story — a requirement serving no story is either
undeclared scope or a missing story, and both need resolving before sign-off.

Split anything containing "and" that could be satisfied by half.

---

## 8. Non-functional requirements

**Every one carries a number and a unit.** The categories below are the minimum set to
consider; omit one only by saying it does not apply.

| Category | Must specify | Example |
|---|---|---|
| Performance | Latency at a percentile, under a stated load | `NFR-1` p99 response under 300 ms at 100 concurrent users |
| Throughput | Rate, with a burst figure | `NFR-2` 500 case updates per minute, bursting to 2,000 |
| Availability | Target, measurement window, planned-maintenance treatment | `NFR-3` 99.5% monthly, excluding announced windows |
| Scalability | The dimension that grows and its ceiling | `NFR-4` 10 branches, 500 concurrent cases per branch |
| Resource bounds | Max size, length, count, depth, timeout, concurrency | `NFR-5` uploads rejected above 20 MB; 30 s request timeout |
| Security | Posture in checkable terms | `NFR-6` all traffic TLS 1.2+; credentials never logged |
| Retention | Duration, and the lawful basis or policy requiring it | `NFR-7` case records retained 7 years, then deleted |
| Accessibility | A conformance level, not an aspiration | `NFR-8` WCAG 2.2 Level AA |
| Observability | What must be answerable in production | `NFR-9` every denial logged with rule id and subject |
| Recovery | RPO and RTO | `NFR-10` RPO 15 minutes, RTO 4 hours |

Rejected phrasings: "fast", "scalable", "secure", "user-friendly", "robust", "highly
available". Each is a category, not a requirement.

---

## 9. MoSCoW prioritisation

Governed by the `sad-moscow` skill. Load it.

---

## 10. Assumptions, questions and risks

```
| ID  | Type       | Statement                              | Owner | Resolution        | If wrong |
|-----|------------|----------------------------------------|-------|-------------------|----------|
| A-1 | Assumption | Branches have reliable broadband       | Client| Confirm per site  | Offline-capable client needed; adds a phase |
| Q-1 | Question   | Third-party hosting, or self-hosted?   | Client| Decide by sign-off| Changes the data-protection position entirely |
| R-1 | Risk       | Paper records incomplete at migration  | Us    | Audit one branch  | Migration scope and cost grow |
```

An assumption with no resolution path is an unmanaged risk — reclassify it. A question with
no owner will not be answered. Never leave a sentence unfinished or a question merely
raised: if it cannot be resolved now, that fact is the entry.

---

## 11. Data model

```
| Entity  | Key fields                   | Owns | Personal data | Retention |
|---------|------------------------------|------|---------------|-----------|
| Client  | name, address, phone, email  | Case | Yes           | 7 years from case close |
| Case    | ref, type, risk band, quote  | —    | Incidental    | 7 years from close |
| Expense | amount, date, category, receipt | — | No           | 7 years from close |
```

Flag personal data explicitly — it determines retention, access control and the lawful
basis, and it is the field most often discovered late. State which entity owns which, so
cascade and deletion behaviour is not left to the implementer.

---

## 13. Interface contracts

A contract is complete or it is absent. Specify every one:

```
POST /api/v1/cases/{caseId}/expenses

Purpose      Record an expense against a case.
Auth         Bearer token. Caller must hold expense:write on the case's branch.
Path params  caseId  uuid, required
Request      { amount: integer (minor units), currency: ISO-4217,
               category: enum[travel, equipment, subsistence, other],
               incurredAt: ISO-8601 date, receiptId: uuid }
Limits       Body max 8 KB. 60 requests/minute per token.
Idempotency  Idempotency-Key header required; replay returns the original 201.

201  { expenseId, caseId, amount, recordedAt }
400  Validation failure. Body names the offending field.
401  Missing or invalid token.
403  Token valid, caller lacks permission on this branch.
404  Case does not exist, or caller may not know that it does.
409  Idempotency-Key reused with a different body.
413  Body exceeds the limit.
422  receiptId references no uploaded receipt.
429  Rate limit exceeded. Retry-After in seconds.
```

Specify 403 and 404 deliberately — whether an unauthorised caller learns the resource exists
is a design decision, not an accident. For events, specify the topic, payload schema, version,
ordering guarantee, delivery guarantee and what a consumer does with a duplicate.

---

## 20. Decision record

One entry per decision that was not obvious, newest first:

```
D-3  Expenses require an uploaded receipt before they can be recorded.
     Date       2026-10-13
     Context    Unreceipted expenses are the loss the client hired us to stop.
     Decision   The API rejects an expense with no receiptId.
     Rejected   Allow recording and flag for later — reproduces the current
                problem in software, and nothing forces the follow-up.
     Costs      Field staff must upload before recording. Mitigated by
                accepting a photo of the receipt.
```

Record what was rejected and why. A decision list without rejected alternatives cannot be
revisited, because the next reader does not know what was already considered.
