# Worked example

An assignment-grade analysis document rewritten to this standard. The source described a
private investigation firm with ten UK branches, no central system, untracked expenses and no
per-branch profitability figures. It ran to several thousand words and contained no
requirements.

The value of the example is the diff, not the prose.

---

## What was cut, and why

| Cut | Reason |
|---|---|
| ~2,000 words defining the SDLC, waterfall phases, Agile, Scrum, Kanban and WIP limits | Methodology exposition. The reader knows, and none of it was a decision about this system. |
| "Functional Testing" and "Non-functional Testing" explained as concepts | Same. The testing *strategy* for this system was never stated — only the definitions. |
| "Before we commence the User Story analysis it is important to know how a user story is typically structured" plus the format explained | The document explained the template and then never used it. |
| The MoSCoW acronym expanded, with the four categories listed and nothing classified | A section defining its own technique instead of applying it. |
| The Kanban-versus-Scrum comparison and conclusion | A delivery-method choice, not a system design decision. Reduced to two lines in section 18. |

Roughly 60% of the original went. Nothing a reader needed to build the system was lost,
because none of the cut text described the system.

## What was missing, and added

| Added | Why it was needed |
|---|---|
| Sections 4, 7, 8, 9, 13, 19 | The document had no scope boundary, no requirements, no numbers, no priorities, no contracts and no definition of success. |
| Won't Have column | Nothing bounded the build. Ten branches and "share files" could mean a document store or a full case-management platform. |
| Acceptance criteria | "They need to share different documents in different folders" is not checkable. |
| Corrected legal basis | See below. |
| Finished assumptions | The original's assumptions section ended mid-sentence, on the single largest open decision in the project. |

---

## The compliance claim, corrected

The original asserted the client was "in breach of the general data protection act of 2018"
because they lose information and cannot calculate profit.

Three errors. There is no "General Data Protection Act 2018" — the instruments are the **Data
Protection Act 2018** and the **UK GDPR**. The **Data Protection Act 1998** was repealed by
the 2018 Act, so citing both as current is wrong. And an inability to calculate profit is a
management-information failure, not a data-protection one; no provision of either instrument
concerns profitability.

What is defensible, stated as a requirement rather than an accusation:

> Loss of client records engages the integrity and confidentiality principle of the UK GDPR
> (Article 5(1)(f)) and the security-of-processing obligation (Article 32). A paper-only
> system with no backup cannot demonstrate compliance with either. `NFR-7` and `NFR-11`
> address this.

The lesson for every SAD: name the instrument, name the obligation, and claim only what the
obligation actually says. An overstated legal claim in a client-facing document is a
liability.

---

## Section 1 — Purpose and context (rewritten)

> Investigate PI operates ten UK branches with no shared system. Case files, client records
> and expense receipts are held on paper at the branch that created them.
>
> Two measurable failures follow. Expenses are reimbursed without evidence, because receipts
> are lost between incurring and claiming — the client reports paying more in expenses than
> expected, with no way to identify which claims were unsupported. And branch profitability
> cannot be calculated, because case revenue sits in one branch's records and the costs
> against it sit in another's.
>
> Both are consequences of the same absence: no single place where a case, its quote and its
> costs are recorded together.

Compare with the original opening, which spent a paragraph on the range of investigations the
firm undertakes — colour that changed no decision.

## Section 6 — User stories (added)

> ```
> US-1  As an intake officer, I need to record an enquiry and produce a quote,
>       so that the client receives a price at first contact.
>
>       AC-1.1  Given an enquiry with a selected risk band, when I request a
>               quote, then the rate for that band is applied and shown.
>       AC-1.2  Given an enquiry missing client contact details, when I try to
>               quote, then it is rejected naming the missing field.
>
> US-4  As a branch manager, I need to see expenses logged against each case,
>       so that I can stop a case exceeding its quoted budget.
>
> US-7  As the owner, I need revenue and cost per branch for a period,
>       so that I can see which branches are profitable.
> ```

`US-7` is the client's stated reason for commissioning the work, and the original document
never captured it as a requirement anywhere.

## Section 9 — MoSCoW (added)

> ```
> | ID    | Requirement                              | Priority | Rationale |
> |-------|------------------------------------------|----------|-----------|
> | FR-2  | Reject an expense with no receipt        | Must     | The loss the client hired us to stop |
> | FR-5  | Case accessible from any branch          | Must     | Removes the single-branch paper file |
> | FR-11 | Revenue and cost report per branch       | Must     | US-7, the commissioning reason |
> | FR-9  | Export expenses to CSV                   | Should   | Accountant can be sent a PDF for one quarter |
> | FR-14 | Per-branch expense trend charts          | Could    | No decision depends on it yet |
> | FR-21 | Mobile receipt capture app               | Won't    | Browser upload from a phone covers it |
> | FR-23 | Case outcome prediction                  | Won't    | Not asked for; would require case history that does not exist |
> ```

`FR-23` is the kind of entry that earns the Won't Have column. Nobody requested it, it is
plausible enough for an implementer to drift toward, and one line stops that.

## Section 10 — Assumptions (finished)

The original ended at "contingencies are in business plans for the clients. A". The decision
it was trailing off into was the largest in the project:

> ```
> | ID  | Type     | Statement                          | Owner  | Resolution         | If wrong |
> |-----|----------|------------------------------------|--------|--------------------|----------|
> | Q-1 | Question | Third-party hosted (Microsoft 365 / | Client | Decide before      | Self-hosting changes cost, the |
> |     |          | Google Workspace) or self-hosted?   |        | sign-off           | DP position and the delivery shape |
> | A-1 | Assumption | All ten branches have reliable    | Client | Confirm per site   | Offline-capable client required; |
> |     |          | broadband                           |        |                    | adds a delivery phase |
> ```

The original made the hosting assumption silently, on cost grounds, in half a sentence. It is
the decision that determines where personal data lives and who processes it, so it is a
question with an owner, not an assumption to be waved through.
