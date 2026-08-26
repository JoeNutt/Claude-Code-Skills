---
name: gtd
description: Runs the user's GTD system, which lives in Evernote. Use for capturing items, clarifying and organising the Inbox, running the daily/weekly/quarterly reviews, maturing tickler items, surfacing next actions and waiting-fors, and drafting status updates or retro notes from what GTD already tracks. Triggers include "capture this", "process my inbox", "clear my inbox", "daily review", "weekly review", "quarterly review", "what am I waiting on", "what's next", "where was I on X", "remind me in September", "draft my status update".
---

# GTD

Evernote is the single system of record. Read and write it through the `evernote` MCP
tools. Never build a parallel markdown copy of the data — a second inbox kills the system.

Read `references/vocabulary.md` before the first Evernote call in a session. It holds the
notebooks, the tag vocabulary, the task tooling, and the query recipes.

## The five steps, and who owns each

| # | Step | Mode | Owner |
|---|---|---|---|
| 1 | Capture | `capture` | agent writes it down, no thinking applied |
| 2 | Clarify | `clarify` | agent proposes what it is and what must happen, user approves |
| 3 | Organise | `organise` | agent proposes destination + tags + dates, user approves |
| 4 | Engage | `next`, `focus`, `draft` | **the user does the work** — agent only surfaces and, on request, drafts |
| 5 | Reflect | `review daily` / `review weekly` / `review quarterly` | agent drives the checklist, user decides |

Step 4 is the user's. Do not do the work of an action unless explicitly told to. Surfacing
the list, opening the project note, or drafting text on request is support; executing the
action is not.

## Modes

### `capture`
Get it out of the user's head and into the **Inbox** notebook. Verbatim, one note per
distinct thing, no interpretation, no tags. Speed over structure. If a dump contains
several unrelated items, split it — but do not refine any of them. That is the next step.

### `clarify`
Answer the two GTD questions — **what is it, and is it actionable?** — and make the note
usable **without filing it**. Rewrite the title to state the thing, restructure the body so
it reads cold in three months, extract commitments in both directions, and for anything
actionable name the **next physical, visible action**.

Propose the rewritten note and the verdict, then wait. Filing is the next step.

### `organise`
Put the clarified item where its kind belongs. Follow `references/routing.md` exactly; the
flow in brief:

- **Not actionable** → propose trash (`delete_note`) · **Reference** + `#reference` ·
  or incubate: `#someday_maybe` when there is no decision date, `#tickler` when there is.
- **Under two minutes** → hand it back to the user to do now. Never filed as a next action.
- **Delegated or blocked** → `#waiting_for`, titled
  `WF: <Name> — <the thing> (asked <YYYY-MM-DD>)`, plus a `create_task` follow-up date.
- **Time-specific** → the calendar, the hard landscape. Never a context action list.
- **Day-specific** → `create_task` with a strict due date, plus `#next_action` + context.
- **ASAP** → `#next_action` + exactly one context tag, no due date.
- **More than one step** → `#gtd_project`, which must carry an outcome statement and
  exactly one live next action, and **only** outcome, state, decisions, and that action —
  support material goes to **Reference** and is linked.

Propose notebook, tags, and any date; apply only after approval.

### `next`
Surface, do not act. List open `#next_action` items, filtered by context tag when the user
names one. Keep it short enough to choose from. Sleeping `#tickler` items never appear
here.

### `focus`
"Where was I on X." Read the `#gtd_project` note plus recent related notes and return the
outcome statement, current state, the last decisions made, and the single next action.
Five lines, not a recap of everything.

### `draft`
Produce text the user will edit and send themselves: a status update, retro notes, a nudge
for a stale waiting-for. Build it from what GTD already holds — notes tagged `#done`,
`search_tasks` completed in the window, open `#next_action` and `#waiting_for` — plus
`git log` when the project is a repo. Never send.

### `review daily` / `review weekly` / `review quarterly`
Run the matching checklist in `references/reviews.md`. Daily opens with the tickler sweep
and then takes the Inbox to zero. Weekly is Sunday, opens with the **mindsweep** — prompt
the trigger list, then wait while the user captures — and clears completed work into
Archive. Quarterly is the Archive purge plus the **Areas of Focus** check.

## Rules for every mode

- **Propose, then apply.** Never file, retag, move, date, or delete without the user
  approving that specific change. Every state transition is theirs. The moment the user
  stops trusting the routing, they stop using the system.
- **Use native Evernote tooling.** Tags via `update_note_tags`, notebooks via `move_notes`,
  and every date, reminder, and completion via `create_task` / `update_task` /
  `search_tasks` / `delete_task`. **Never create, complete, or modify a task by splicing
  ENML** — `edit_note` is for note content only.
- **The two-minute rule is a hand-back, not a filing.** Say "do it now" and let the user do
  it. Tracking a two-minute action costs more than doing it.
- **A project without a next action is broken.** So is one without an outcome statement, or
  one carrying two live next actions. Fix on sight.
- **The calendar is sacred.** Only what genuinely must happen that day at that time.
  Aspirational work on the calendar destroys its credibility.
- **Never send anything.** Drafts are handed back, not delivered.
- **Facts about people, never judgments.** See the read-aloud test in
  `references/routing.md`. Applies to note bodies, titles, and waiting-for labels.
- **Trash is as far as you go.** `delete_note` moves a note to trash, which is
  recoverable. Emptying the trash is the user's job in the Evernote app. Never present a
  deletion as permanent and never batch-delete without an itemised list approved first.
- **One context tag per action.** Pick the one that determines where or how it can be done.
- **Personal counts.** Home, shopping, and household items go through the same pipeline
  with `@home`. GTD covers everything or it covers nothing.
