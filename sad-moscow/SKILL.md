---
name: sad-moscow
description: This skill should be used when prioritising requirements, scoping a release, deciding what makes a delivery and what is deferred, or filling in section 9 of a systems analysis and design document. It carries the four MoSCoW categories and the test for each, the effort ceiling on Must Have, the classification table format, and the failure modes - everything marked Must, an empty Won't Have, priorities treated as permanent. Trigger phrases include "MoSCoW", "prioritise these requirements", "prioritize the backlog", "must have", "should have", "could have", "won't have", "what's in scope for this release", "what can we cut", "is this a must or a nice to have", "scope this delivery", "trim the scope", "what are we not building".
---

# MoSCoW prioritisation

Sorts requirements into what a delivery **must** contain, what it **should**, what it
**could**, and what it explicitly **will not** this time. The output is a table in which every
requirement is classified — not a discussion of priorities.

Within a `sad` document this is section 9. It is used independently whenever a release is
scoped or a backlog is trimmed, which is why it is a separate skill.

## The four categories

| Category | Meaning | The test |
|---|---|---|
| **Must Have** | Without it the delivery has failed. Legal, safety, or the reason the work was commissioned. | Would you ship without it? If yes, it is not a Must. |
| **Should Have** | Important and painful to omit, but a workaround exists for one delivery cycle. | Name the workaround. If you cannot, it is a Must. |
| **Could Have** | Desirable, little impact if dropped. The first thing traded away. | Is anyone blocked by its absence? If yes, it is higher. |
| **Won't Have** | Could be built, deliberately is not being built **this time**. | Is it written down? If not, it will be built by accident. |

It is **Won't Have (this time)** — never "Cannot Have". The distinction is the whole point:
these are achievable things being consciously deferred, which is why they are recorded rather
than omitted. "Cannot" describes an impossibility and needs no prioritisation decision.

## Must Have has a ceiling

**Must Have should not exceed roughly 60% of estimated effort.**

If nearly everything is a Must, no prioritisation has occurred and there is no contingency
left to trade when the delivery runs late — which it will. The remaining 40% in Should and
Could is the buffer that lets a date hold without a renegotiation.

When a stakeholder marks everything Must, the question is not "are you sure?" but **"if the
date were at risk, which of these would you ship without?"** The answer re-sorts the list.

## The output

```
| ID    | Requirement                              | Priority | Rationale |
|-------|------------------------------------------|----------|-----------|
| FR-2  | Reject an expense with no receipt        | Must     | The loss the client commissioned this to stop |
| FR-5  | Case accessible from any branch          | Must     | Removes the single-branch paper file |
| FR-9  | Export expenses to CSV                   | Should   | Accountant accepts a PDF for one quarter |
| FR-14 | Per-branch expense trend charts          | Could    | No decision depends on it yet |
| FR-21 | Mobile receipt capture app               | Won't    | Browser upload from a phone covers the need |
| FR-23 | Case outcome prediction                  | Won't    | Not requested; needs case history that does not exist |
```

**Rules**

- **Every requirement is classified.** An unclassified requirement is an unmade decision that
  surfaces as an argument during delivery.
- **Rationale is mandatory, and names a consequence** — not "high priority", which restates
  the category. A Must whose rationale is "the client wants it" has not been tested.
- **Won't Have is populated.** An empty column means scope was never bounded.
- **Classify requirements, not features.** `FR-2` and `NFR-5` are classifiable because they
  are single testable statements. "Reporting" is a topic and cannot be prioritised.
- **A priority is per delivery, not permanent.** Re-sort at each delivery boundary; a Won't
  Have from last cycle is a candidate Must in this one. Record the cycle the table applies to.

## Won't Have is the most valuable column

It is the only part of the document that says **stop**, and it is the one most often left out.

This matters most when the implementer works at volume. An agent given a requirements
document with no stated exclusions will fill the space with what seems reasonable — `FR-23`
above is exactly that shape: nobody asked for case outcome prediction, it is plausible enough
to drift toward, and one line prevents it. Unbounded scope gets filled.

Populate it with the adjacent things a reasonable implementer might assume: the obvious next
feature, the integration that seems implied, the admin screen nobody requested.

## Failure modes

| Symptom | What has gone wrong | Fix |
|---|---|---|
| Everything is Must | No prioritisation happened | Ask which would ship late if the date slipped |
| Won't Have is empty | Scope unbounded | List what a reasonable implementer might add unasked |
| Rationale restates the priority | The classification was never justified | Name the consequence of omitting it |
| Topics in the ID column | Features, not requirements | Classify numbered `FR-n` / `NFR-n` entries |
| Priorities unchanged across three cycles | The table is decoration | Re-sort at each boundary |
| A Should Have with no workaround | It is a Must | Reclassify, and face the effort total |
