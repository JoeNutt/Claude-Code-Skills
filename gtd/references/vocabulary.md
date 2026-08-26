# Vocabulary

The live structure of the user's Evernote GTD system. Verified 2026-08-26.

## Notebooks

| Notebook | Holds |
|---|---|
| **Inbox** | Raw captures, unclarified. Target state is empty. |
| **Actions** | Everything live — next actions, projects, waiting-fors, ticklers, someday/maybe. |
| **Reference** | Non-actionable material worth keeping, plus all project support material. Retrieval only, no review obligation. |
| **Archive** | Completed and dead items. Cleared into weekly, purged quarterly. |

Legacy, do not file anything new here:

- **=** — an old catch-all dump. Items surface from it during quarterly review.
- **(Imported) 12WY_Template**, **(Imported) 12WY_Template_1** — 12 Week Year templates.

## Tags

**Context** — exactly one per action, whichever determines where or how it can be done:

`@work` · `@coding` · `@research` · `@testing` · `@home`

`@work` is the default for professional items; use `@coding`, `@research`, or `@testing`
when the action is specifically at a keyboard writing code, reading up on something, or
verifying something, since those are the modes the user actually batches by.

**State** — the GTD list a note belongs to:

| Tag | Meaning |
|---|---|
| `#next_action` | A single physical, visible action that can be started now. No due date unless genuinely day-specific. |
| `#gtd_project` | Any desired outcome requiring more than one physical action step, completable within a 12-month window. Must carry an explicit outcome statement and exactly one live `#next_action`. |
| `#waiting_for` | Delegated or blocked on someone else. Label format and follow-up task in `routing.md`. |
| `#tickler` | Asleep until a known trigger date. Carries a task due date or note reminder; carries no `#next_action`. Matures in the daily review. |
| `#someday_maybe` | Not committed to, no decision date. Reviewed as a batch weekly. |
| `#done` | Completed. Cleared into Archive at the weekly review. |
| `#review` | Flagged for a decision at the next review. |
| `#reference` | Non-actionable, kept. |

`#tickler` was created 2026-08-26 and is flat, alongside the other state tags.

**Modifiers:** `#high` (priority) · `#jira` (has a Jira ticket) · `#12WY` (serves a current
12 Week Year goal) · `WeeklyReview` (notes produced by a weekly review).

**Legacy duplicates** — do not apply, consolidate when encountered:
`Needs Review` → `#review` · `Research` → `@research` · `12WY` → `#12WY`.

## Areas of Focus (Horizon 2)

A single note in **Reference**, titled **Areas of Focus**, listing the ongoing roles and
responsibilities the user holds — the standards to be maintained rather than outcomes to be
reached. Typically things like *Health*, *Lead Developer*, *Finances*, *Home*, *Family*,
*Professional development*.

The distinction that makes it useful: **an area is never completed.** "Get fit" is a
project; "Health" is an area. Areas have no next action and no due date — they exist to be
checked *against*, so a responsibility with nothing moving under it becomes visible.

It is reviewed at the **quarterly** review, not weekly — Horizon 2 changes slowly, and
checking it too often turns it into noise. See `reviews.md`.

If the note does not exist yet, offer to draft one from the roles the user actually holds.
Never invent areas on their behalf.

## The calendar is not a tag

Time-specific actions live on the calendar — an Evernote Calendar note or a task carrying
that exact time — and never on a context action list. The calendar is the **hard
landscape**: only what genuinely must happen that day at that time. Nothing aspirational
goes there. Day-specific-but-not-timed work is a task due date instead, not a calendar
entry.

## Dates and reminders are Evernote Tasks

Every due date, reminder, and trigger in this system is an **Evernote task**, created and
changed through the task tools. Tasks live inside notes and carry due date, priority, flag,
and recurrence.

- `create_task` — add a task to a note, with the due date as the mechanism for waiting-for
  follow-ups, day-specific deadlines, and tickler trigger dates.
- `update_task` — change any field, **including marking it complete**. Completion is an
  `update_task` field, not a note edit.
- `search_tasks` — query by `dueDate` range, `status`, `priority`, `flagged`. This is how
  mature ticklers and due follow-ups are found.
- `delete_task` — remove a task.

**Never create, complete, or modify a task by editing ENML.** Tasks appear in a note's body
only as a placeholder; the real entities come back in `structuredContent.tasks[]` from
`get_note`, and splicing the body will not change them.

Likewise use `update_note_tags` to change tags and `move_notes` to change notebooks —
never hand-edit those through the body either. `edit_note` is for note *content* only.

## Query recipes

`search_notes` uses Evernote grammar. Pass `clientTimeZone: "Europe/London"` whenever the
query contains a date-relative operator.

```
notebook:"Inbox"                                  everything awaiting clarify
tag:"#next_action" -tag:"#done"                   the live action list
tag:"#next_action" tag:"@coding"                  actions by context
tag:"#waiting_for" -tag:"#done"                   the waiting-on list
tag:"#gtd_project" -tag:"#done"                   live projects
tag:"#tickler" -tag:"#done"                       everything currently asleep
tag:"#done" updated:day-7                         completed this week
tag:"#someday_maybe"                              the maybe pile
notebook:"Archive" -updated:month-3               archive older than a quarter
tag:"#12WY" -tag:"#done"                          work serving the 12-week goals
```

Date-driven lookups go through `search_tasks`, not note grammar:

```
search_tasks status:open dueDate:{to: <end of today, ISO with offset>}
    → everything due or overdue: mature ticklers, due follow-ups, day-specific actions.
      Cross-reference the returned noteId against tag:"#tickler" to isolate ticklers.

search_tasks status:open dueDate:{from: <today>, to: <today + 7d>}
    → the week ahead, for the weekly review.

search_tasks status:completed dueDate:{from: <last review>, to: <now>}
    → what actually got finished, for the weekly update draft.
```

`dueDate` bounds are ISO-8601 and **must carry an offset**. For completed tasks the range
filters by completion date, not due date.

Use `semantic_search` when the user describes a note by meaning rather than by tag or
title. Use `search_notes` for anything list-shaped.

## Known state

As of 2026-08-26 the Inbox is empty and **Actions is acting as the real inbox** — it mixes
live work notes with unclarified personal captures (shopping lists, a Zoopla listing, a
watchlist, a soundbar basket). The first-run sweep in `reviews.md` addresses this.
