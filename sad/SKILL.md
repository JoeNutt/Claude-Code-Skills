---
name: sad
description: This skill should be used when producing the document that defines what will be built and why - a systems analysis and design document, design doc, technical plan, PRD, requirements specification or scoping document - and when publishing such a plan as an artifact. It covers requirements elicitation, user stories, MoSCoW prioritisation, functional and non-functional requirements, assumptions and risks, interface contracts, success criteria, and the structure the finished document takes. Trigger phrases include "systems analysis and design", "SAD", "design document", "design doc", "write up a plan", "technical plan", "PRD", "requirements document", "requirements spec", "spec this out", "scope this", "user stories", "MoSCoW", "acceptance criteria", "non-functional requirements", "write this up before we build", "publish this plan", "what are we actually building".
---

# Systems Analysis and Design

The document that settles **what is being built, for whom, and what it must do** — before
anything is built. It is implemented from and signed off against, so every line either
states a requirement, records a decision, or defines a contract.

This is not architecture. Component decomposition, dependency direction and layering belong
to `architecture`; this skill governs **requirements and the document itself**.

## The one rule

**A SAD records requirements, decisions and contracts. It never explains methodology.**

Never define what Agile is, what a user story is, what MoSCoW stands for, what functional
and non-functional testing are, or the phases of the waterfall model. A professional reader
knows, and every paragraph of exposition is a paragraph not spent on a decision. This is the
most common way these documents become unreadable, and then unread.

The test: **if a sentence would be equally true of someone else's project, delete it.**

## Why the rigour pays

A numbered requirement with acceptance criteria is a contract an implementer can be held to.
A paragraph of prose is an invitation to invent scope. When the implementer is an agent
working at volume, that difference stops being stylistic:

- **User stories with acceptance criteria** define done in terms a reviewer can check,
  rather than something that gets declared finished.
- **A populated Won't Have list** is the only part of the document that says *stop*. It is
  the most frequently omitted section and the most valuable one — unbounded scope gets
  filled.
- **Numbered, traceable requirements** make it answerable whether a given piece of work was
  ever asked for.

Write for a reader who will not ask a clarifying question.

## Scale to the work

Three tiers. Choose on first contact and say which you are writing. Padding a small change
into a full specification is as much a defect as under-specifying a new system.

| Tier | When | Sections |
|---|---|---|
| **Lite** | A change inside an existing system; one component; no new external contract. | 1, 4, 10, 12, 16, 17, 20 |
| **Standard** | A new feature, service, or endpoint. The default. | Lite + 2, 5, 6, 7, 8, 9, 13, 19 |
| **Full** | A new system, a new product, or anything with external stakeholders or regulatory exposure. | All of them |

## Sections

Numbering is fixed so documents stay comparable. Omit by tier, never renumber.

| # | Section | Must contain |
|---|---|---|
| 1 | Purpose and context | The problem, whose problem it is, why now. No solution. |
| 2 | Current state | How the work is done today and precisely where it fails. |
| 3 | Consequences of inaction | What happens if this is not built. Cost, risk, exposure. |
| 4 | Scope and non-goals | What is in, and an explicit list of what is deliberately not being built. |
| 5 | Stakeholders and users | Each role, what they need from the system, what they can already do. |
| 6 | User stories | Stories in the fixed format, each with acceptance criteria. Load `sad-user-stories`. |
| 7 | Functional requirements | Numbered `FR-n`, each traceable to a story. |
| 8 | Non-functional requirements | Numbered `NFR-n`, **each carrying a number and a unit**. |
| 9 | MoSCoW prioritisation | A table classifying every FR and NFR. Load `sad-moscow`. |
| 10 | Assumptions, questions and risks | Each with an owner, a resolution path, and the impact if wrong. |
| 11 | Data model | Entities, relationships, ownership, retention, and what is personal data. |
| 12 | Design | How it works. Defer code structure to `architecture`. |
| 13 | Interface contracts | Every endpoint, event and command, fully specified. |
| 14 | Front-end design | Flows and every state: loading, empty, error, partial, denied. |
| 15 | Security and compliance | Trust boundaries, authorisation model, named legal instruments. |
| 16 | Failure modes and degradation | What fails, how it is detected, whether it fails open or closed. |
| 17 | Testing strategy | How each requirement is verified, and by what kind of test. |
| 18 | Delivery approach | Phasing and milestones. Methodology only if the choice is live. |
| 19 | Success metrics | Numbers that decide whether this worked. |
| 20 | Decision record | Decisions taken, alternatives rejected, and why. |
| 21 | References | Real sources for load-bearing external claims. |

Formats and worked templates for sections 7, 8, 10-13 and 20 are in
`references/sections.md`. Read it before writing those sections, not after. Sections 6 and 9
have their own skills — load those rather than working from memory.

## Hard rules

- **Every requirement is numbered and traceable.** `FR-4` must name the story it serves.
  An untraceable requirement is a preference.
- **Every non-functional requirement carries a number and a unit.** "Must be fast" is not a
  requirement. "p99 under 200 ms at 50 requests per second" is. No number means no test,
  which means it will not hold.
- **Every assumption names an owner and how it gets resolved**, plus what changes if it is
  wrong. An assumption with no resolution path is an unmanaged risk.
- **Sections 6 and 9 are governed by `sad-user-stories` and `sad-moscow`.** Their rules live
  there and are deliberately not restated here, so there is one place to change them.
- **Interface contracts are complete or absent.** A partial contract is worse than none,
  because it gets implemented. Specify auth, errors and limits, not just the happy path.
- **Compliance claims name the instrument.** "GDPR" alone is not a citation. Name the Act or
  Regulation and the specific obligation. Never assert a breach that is not one.
- **No section explains itself.** A section that defines its own technique instead of
  applying it is empty and must be rewritten or cut.
- **Resource bounds are requirements, not implementation detail.** Size, rate, depth,
  concurrency and timeout belong in section 8 with numbers.
- **State what is not known.** An open question recorded is a finding; one omitted is a
  defect discovered later at cost.

## Writing standards

Present tense, active voice, specific nouns. Say "the control plane rejects the request"
rather than "requests may potentially be rejected by the system".

Tables for anything enumerable — requirements, contracts, states, risks. Prose only where a
mechanism needs explaining.

Delete: throat-clearing openings, restatements of the section heading, "it is important to
note that", summaries of what the document will go on to say, and any passage whose purpose
is to demonstrate the author's knowledge.

Cite real sources with real URLs. One good citation beats five padding ones, and a fabricated
reference discredits the whole document.

## Publishing

A finished SAD is a deliverable with an audience, so it is published as an artifact rather
than left in a terminal or a local file. Load `artifact-design` before writing the page, and
`artifact-diagramming` when the design or data model needs a diagram.

Keep the section numbering in the published version. A reader quoting "NFR-7" must be able to
find NFR-7.

## Where another skill owns the decision

Name the decision in the SAD; make it under the relevant standard.

| Section | Owned by |
|---|---|
| 6, stories and acceptance criteria | `sad-user-stories` |
| 9, prioritisation and scope boundary | `sad-moscow` |
| 12, component structure and dependency direction | `architecture` |
| 15, threat model and trust boundaries | `sec-threat-modeling` |
| 15, authorisation model | `sec-authz-least-privilege` |
| 8 and 11, validation and limits at the boundary | `sec-input-validation` |
| 17, test strategy and what gets written first | `tdd` |
| 14, visual design and interaction | `frontend-design` |
| 1 and 3, wording for a non-technical audience | `cc2-stakeholder-communication` |

`references/worked-example.md` shows an assignment-grade document rewritten to this
standard, with the cuts and additions called out.
