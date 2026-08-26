# Clarify and Organise

Two separate steps, in Allen's order. **Clarify** decides what a thing is and what, if
anything, must happen about it. **Organise** puts the result where its kind belongs.
Propose each before applying.

---

# Clarify

## Step 1 — Make the note usable

A raw capture is not yet usable. Transform it so it survives being read cold in three
months by someone with no memory of the moment it was written.

1. **Title states the thing, not the source.** "Sony Bravia Theatre Bar 8 + 55in OLED
   bundle" not "johnlewis.com". "Networking issues to raise re: paul-bot fix" not "notes".
2. **Body becomes readable.** Summarise walls of pasted text. Keep the specifics — repo
   names, file paths, error strings, prices, dates, people. Strip ad copy, signatures,
   nested quoting, stock counts, tracking parameters in URLs.
3. **Extract commitments in both directions.** "I said I'd send Eric the deployment doc" is
   a next action. "Dan is rotating the New Relic key" is a waiting-for. Each becomes its
   own item, even when they arrived in one dump.
4. **Flag ambiguity** rather than guessing.

Never discard detail on the assumption it is noise — summarise above it, or ask before
cutting anything the user wrote themselves.

## Step 2 — What is it, and is it actionable?

The two questions that drive everything below. Answer them explicitly for each item.

---

## Not actionable

Three outcomes, and nothing else. Do not park unactionable things on an action list.

| Outcome | Where | Meaning |
|---|---|---|
| **Trash** | propose `delete_note` | No value, no reference use, no future interest. Recoverable via trash. |
| **Reference** | **Reference** notebook + `#reference` | Non-actionable, worth keeping. Filing is retrieval-only — no review obligation. |
| **Incubate** | see below | Might matter later, but not committed to now. |

**Incubate splits two ways** — this distinction matters and is easy to get wrong:

- `#someday_maybe` — no decision date. Reviewed as a batch each week. Use when the item
  needs a *judgment* later ("learn Rust", "replace the sofa").
- `#tickler` — a *known* date when it should reappear. Use when the item needs to be
  *invisible until* a specific day ("chase the warranty claim on 12 Sep", "renew domain in
  March"). See the tickler section below.

If you can name the date it should resurface, it is a tickler, not a someday/maybe.

---

## Actionable

First name the **next physical, visible action** — what a camera could record you doing.
"Think about the deployment doc" is not one; "Draft the deployment doc outline in Notion"
is. Then work the ladder. First match wins.

### 1. Under two minutes → do it now

If the next action takes less than two minutes, **hand it straight back to the user to
execute now**. Do not file it, do not tag it `#next_action`, do not create a task — the
overhead of tracking it exceeds the cost of doing it.

Say plainly: *"This is a two-minute item — do it now."* Once the user confirms it is done,
tag `#done`; it clears to Archive at the weekly review.

### 2. Delegated or blocked on someone → `#waiting_for`

→ **Actions** + `#waiting_for` + one context tag. Title format, exactly:

```
WF: <Name> — <the thing> (asked <YYYY-MM-DD>)
```

Then `create_task` on that note with a **due date set to the follow-up date** — when *you*
will chase, not when they promised. That reminder is what makes the list self-policing
instead of something you have to remember to read.

Record only facts: what was asked, of whom, when. Nothing about why it might slip.

### 3. Time-specific → the calendar

An action that must happen **at a particular time** — a meeting, a call, a deploy window —
goes on the **calendar**, via an Evernote Calendar note or a task carrying that exact time.
Never onto a context action list.

The calendar is Allen's **hard landscape**: it holds only what genuinely must happen on
that day at that time. Nothing else goes there, ever. The moment aspirational work leaks
onto the calendar, it stops being trustworthy and you start ignoring it.

### 4. Day-specific → task with a strict due date

Must happen **on** a given day but at no particular time — submit by Friday, call while
they are in the office Tuesday. Use `create_task` on the note with that due date, and
`#next_action` plus one context tag so it also appears on the right list.

A real deadline only. **A wish is not a deadline** — if the date is aspiration rather than
consequence, it belongs in category 5 with no date at all. Fake due dates are the fastest
way to make the due-date view worthless.

### 5. As soon as possible → `#next_action`

Everything else. → **Actions** + `#next_action` + **exactly one** context tag:

`@work` · `@coding` · `@research` · `@testing` · `@home`

Pick the context that determines *where or how* the action can be done, since that is how
the user batches. No due date. Title starts with a verb and names a physical action:
"Email Eric the deployment strategy doc", not "Deployment doc".

---

## Projects

> **A project is any desired outcome that requires more than one physical action step and
> can be completed within a 12-month window.**

Note what this does and does not mean:

- **More than one step.** Two actions makes it a project. Size is irrelevant — "replace the
  kitchen tap" is as much a project as a quarter of platform work.
- **Within 12 months.** Anything genuinely beyond a 12-month horizon is not a project. It
  is a goal or an area of focus — park it as `#someday_maybe` and let the weekly review
  decide when it becomes live.

Every `#gtd_project` note in **Actions** must carry both of these, without exception:

1. **An explicit outcome statement** at the top of the note, answering *what does done look
   like?* in one sentence, phrased as a completed state: "The quota-request reusable
   workflow is merged and the boss has signed off." Not a topic. Not a title restated.
2. **Exactly one live `#next_action`**, tracked either as an Evernote task inside the
   project note (`create_task`, preferred — it keeps the action with its outcome) or as a
   separate `#next_action` note that names the project in its title.

A project with no next action is the system's most common silent failure: it looks tracked
while nothing is moving. Never leave one in that state, and never let a project carry two
live next actions — pick the one that comes first.

### The project list is not the project's filing cabinet

Allen keeps the project *list* and the project *support material* physically apart, and the
same separation has to hold here or the list stops being reviewable.

**The `#gtd_project` note in Actions holds four things and nothing else:**

1. the outcome statement,
2. current state — a few lines on where things stand,
3. decisions made, with their reasoning,
4. the link to, or task for, the single live next action.

**Everything else goes to Reference and is linked from it.** Meeting notes, specs, research,
transcripts, screenshots, PDFs, diagrams, exported logs, long email threads, vendor
documentation, anything with attachments — file it in **Reference**, tag it `#reference`
plus whatever makes it findable, and leave a link in the project note under a short
*Support material* heading.

The reason is mechanical, not aesthetic. The weekly review walks every project note; if
those notes are twenty screens of pasted meeting minutes, the review becomes unreadable and
gets skipped, which is how the whole system dies. A project note should be scannable in
about fifteen seconds.

When organising an item that clearly belongs to an existing project, ask which it is: a
*next action*, a *decision or state change* to fold into the project note, or *support
material* for Reference. Those three routes cover almost everything, and the answer is
support material more often than people expect.

Keeping the note lean is also what lets `focus` answer "where was I" from the project note
alone, without re-reading a quarter of accumulated material.

---

## Modifiers

Apply on top of the above:

- `#high` — actually urgent, not merely important. If everything is high, nothing is.
- `#jira` — a Jira ticket exists; put the key in the title.
- `#12WY` — serves a current 12 Week Year goal. Be honest about this: the weekly review
  checks what proportion of committed work is actually pointed at the goals.

---

## The digital tickler

Allen's tickler is 43 physical folders — 31 for the days of this month, 12 for the months
ahead — that you empty one per day, so a thing you cannot act on yet disappears until the
morning it matters.

**In Evernote the date replaces the folder.** There are no 43 containers; there is a tag
and a due date.

To put something to sleep:

1. Leave the note in **Actions** (or **Inbox**), tagged `#tickler`.
2. `create_task` on the note with the due date set to the **trigger date** — the day it
   should reappear. A note-level reminder works equally well for a whole-note tickler.
3. Strip any `#next_action` tag. A sleeping item is not on an action list; that is the
   entire point.

The 12 month-folders map to a trigger date on the **1st of that month**, at which point the
item resurfaces and gets re-decided — usually into a real date, or back to sleep.

Ticklers mature during the **daily review**, which is the equivalent of emptying today's
folder. See `reviews.md`.

Good candidates: a chase that would be premature today, a renewal, something waiting on an
external date, a decision deliberately postponed to a day you have chosen.

---

## The read-aloud test

Before writing any line that mentions a colleague, ask: could this be read aloud to them?

Write facts and stated preferences — what was asked, when, what was agreed, that someone
prefers the risk stated up front. Never write assessments of reliability, competence, or
motive, and never record office politics. If a capture contains that, clarify it into the
underlying fact or leave it out, and say which you did.

This is not squeamishness. These notes name identifiable people, sync to a cloud account,
and the pattern of repeat follow-ups on a waiting-for tells you everything an opinion
would — while staying something the user could hand to anyone.

## People notes

Create a Reference note for a colleague only when a `draft` has come out wrong for want of
it, and keep it to **communication preferences only**: format, level of detail, what they
want to hear first. Not a profile, not a history, not a project map.
