---
name: cc2-stakeholder-communication
description: This skill should be used when writing about technical work for a non-technical audience - release notes, status updates, product announcements, executive summaries, client emails, incident notices, or explaining an architectural change to someone who does not read code. It replaces implementation jargon with plain language and business consequence. Trigger phrases include "explain this to the client", "draft a product update", "write a summary for management", "release notes", "status update", "explain this to a non-technical", "write this up for stakeholders", "customer-facing", "exec summary", "what do I tell them".
---

# Communicating Technical Work to Non-Technical Audiences

Match the vocabulary to the audience. The reader needs to make a decision, set an
expectation, or feel informed — none of which require them to understand the implementation.

The test for every sentence: **could a smart reader who has never seen the codebase act on
this?** If not, rewrite it.

## Lead with consequence, not mechanism

The reader's first question is "what does this mean for me?" Answer it first; supply the
mechanism only if it changes their decision.

| Instead of | Write |
|---|---|
| "Refactored the auth middleware to eliminate a race condition in token refresh" | "Fixed the intermittent logouts some users hit during long sessions" |
| "Migrated from REST polling to WebSockets" | "Updates now appear instantly instead of on a 30-second delay" |
| "Added database indexes on the orders table" | "The order history page now loads in under a second, down from about eight" |
| "Introduced a circuit breaker on the payments client" | "A payment provider outage no longer takes the checkout page down with it" |
| "Increased unit test coverage to 80%" | "We now catch this category of problem before release rather than after" |

Note what each rewrite does: it names who is affected, what changes for them, and — where
possible — a number they can hold on to.

## Use plain language, and metaphor where it earns its place

Software work maps well onto construction: foundations, load-bearing structure, wiring,
scaffolding, shoring something up before building on top of it. A metaphor is useful when it
lets the reader reason a step further on their own — "we're replacing the foundation before
adding another floor" conveys both the necessity and the lack of visible progress.

Keep it honest. A metaphor that oversells the drama, or implies a permanence the work does
not have, costs credibility the next time. Drop the metaphor entirely once it stops carrying
weight; stacking three of them is worse than using none.

Words to keep out unless the audience genuinely shares them: refactor, polymorphism, race
condition, middleware, dependency injection, technical debt, idempotent, cache invalidation,
CI/CD pipeline, schema migration, N+1 query. Each has a plain-language equivalent that
communicates more.

## Be honest about cost and risk

This is where technical writing for executives most often goes wrong, in both directions —
burying a risk to avoid a difficult conversation, or inflating one to justify a timeline.

- Give real estimates, including the uncertainty. A range with a reason beats a false point
  estimate: "two to three weeks, depending on whether the vendor's API supports bulk export."
- Name trade-offs plainly: what was gained, what was given up, what was deferred.
- Say what is not fixed. A status update that implies completeness it does not have is the
  expensive kind of wrong.
- Where work has no visible output — a rewrite, a migration, hardening — say so directly and
  explain what it buys. "Nothing looks different, and that's expected; this removes the
  cause of the three outages last quarter."
- Do not agree that something is simple when it is not, and do not manufacture complexity to
  make routine work sound impressive.

## Format for the audience

- **Release notes:** grouped by user-visible impact (new, improved, fixed). One line each.
  Omit anything with no user-facing consequence.
- **Status update:** what shipped, what is in progress, what is blocked and what would
  unblock it, what changed since last time. Blockers get a named owner and an ask.
- **Executive summary:** the decision or headline in the first sentence. Detail below, in
  case they read on. Assume they may read only the first line.
- **Incident notice:** what happened, who was affected and how, whether it is resolved, what
  prevents a recurrence. No blame, no unnecessary internal detail, no speculation before the
  facts are in.
- **Client email:** their situation first, then what you did, then what they need to do.

Keep it short. Length reads as either padding or bad news, and a stakeholder update is
usually skimmed rather than read.

## Before sending

- Would the reader know what to do, or what to expect, after one read?
- Is every term one they use themselves?
- Are the claims true, including the implied ones — is anything overstated by omission?
- Have you said what is *not* done?
- Would you be comfortable if this were forwarded, unedited, to the person it describes?
