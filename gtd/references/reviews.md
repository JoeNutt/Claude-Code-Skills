# Reviews

Three cadences, three different jobs. Run the checklist; the user makes every call.

---

## Daily — each morning

**Job: empty today's tickler folder, then take the Inbox to zero.** Nothing else. Do not
drift into reorganising projects.

### 1. Tickler sweep — before anything else

This is the digital equivalent of pulling today's folder from the 43-folder file. Do it
first, because mature ticklers become Inbox items that this same pass then processes.

1. `search_tasks` with `status: "open"` and `dueDate: {to: <end of today, ISO+offset>}`.
2. Keep the hits whose note carries `#tickler` — cross-reference against
   `search_notes` `tag:"#tickler" -tag:"#done"`. Also catch any `#tickler` note whose
   trigger is a note reminder rather than a task.
3. For each mature item, propose:
   - remove `#tickler` via `update_note_tags`,
   - `move_notes` it into **Inbox**,
   - `update_task` to complete the trigger task (it has done its job).
4. Apply on approval, then present them at the front of the Inbox pass below. A matured
   tickler is raw input again — it gets clarified and organised like anything else, and it
   may well go straight back to sleep with a new date.

Ticklers **not** yet due stay invisible. Do not list them, do not mention them, do not
"just check" what is coming — that defeats the mechanism.

### 2. Inbox to zero

1. List `notebook:"Inbox"`.
2. For each note, run **clarify** then **organise** (see `routing.md`). Present one item at
   a time with the proposed rewrite, destination, and tags. Apply on approval.
3. **Two-minute items:** say so and hand them straight back to the user to do now. They do
   not get filed as next actions.
4. Anything that cannot be decided in a few seconds: `#review`, and move on. Do not let one
   ambiguous item stall the pass.
5. Close by stating the count cleared, the count deferred to `#review`, and the number of
   ticklers that matured.

If both the tickler queue and the Inbox are empty, say so and stop. Do not invent work.

---

## Weekly — Sunday

**Job: get current and get honest.** Roughly fifteen minutes with the checklist driving.

### Step 0 — The mindsweep

**Do this before touching the Inbox.** Everything downstream only works on what has been
captured, so the review starts by emptying the user's head — not their Inbox.

Prompt with a short trigger list and let it jog loose whatever is lingering. Vary which
categories you lead with week to week so it does not become wallpaper:

> Anything on your mind about — **work:** boss, team, current projects, code review debt,
> people you owe a reply, upcoming meetings, on-call. **Personal:** health, family, pets,
> the house, anything broken or half-fixed. **Admin:** finances, bills, renewals, software
> and domain subscriptions, insurance, travel coming up. **Loose:** things you keep meaning
> to look up, learn, buy, or cancel.

Then **stop and wait.** The user captures whatever surfaced into the **Inbox** notebook —
via `capture` or on their own. Do not start clarifying, do not proceed to step 1, and do
not offer to guess what might be on their mind. An unhurried silence here is the point;
rushing it produces an empty sweep and a review that reviews nothing new.

When they say they are done, note how many items landed and continue.

### The checklist

1. **Inbox to zero** — run the daily pass, tickler sweep included, over everything sitting
   there including the mindsweep captures.
2. **Clear completed** — `tag:"#done"` still in Actions → move to **Archive**. Report the
   count; this is the week's evidence of delivery and feeds step 9.
3. **The week ahead** — `search_tasks` for `dueDate` between today and +7 days. Surface
   what is landing: calendar commitments, day-specific actions, follow-ups, and ticklers
   about to mature. This is the one time it is legitimate to look at sleeping ticklers.
4. **Waiting-for aging** — `tag:"#waiting_for" -tag:"#done"`. Anything asked more than
   seven days ago: still needed? If yes, draft a nudge for the user to send and push the
   follow-up task's due date out with `update_task`. Past fourteen days needs a decision —
   chase, escalate, or drop it.
5. **Project integrity** — for each `tag:"#gtd_project" -tag:"#done"`, confirm both
   requirements hold: an explicit outcome statement, and **exactly one** live
   `#next_action`. List every project that fails either test and fix them here. Also flag
   any project whose realistic completion is beyond a 12-month horizon — that is a goal,
   not a project, and belongs in `#someday_maybe`.
6. **Stale next actions** — `#next_action` untouched fourteen days or more. Still real, or
   delete? A list nobody believes is worse than no list.
7. **Someday/maybe** — surface five from `#someday_maybe` at random. Promote to a project,
   give it a tickler date, leave, or drop.
8. **12WY check** — compare the live `#next_action` list against the current 12 Week Year
   goals. State plainly what proportion of committed work serves them. This is the step
   that makes the system worth running; do not soften the answer.
9. **Draft the week's update** — from the notes archived in step 2 plus `search_tasks`
   completed since the last review, using `draft`. Hand it back for editing.
10. Write the review's output as a note in **Actions** tagged `WeeklyReview`, so the next
    review and the quarterly one have a trail.

---

## Quarterly — Archive purge

**Job: shrink the Archive so search stays useful.** Destructive, so move slowly.

1. List `notebook:"Archive" -updated:month-3`, plus anything remaining in the legacy `=`
   notebook.
2. Sort into three proposed piles and present them **itemised** — never a bulk count:
   - **Keep** — records with future value: decisions and their reasoning, shipped work
     worth citing, anything with legal, financial, or employment relevance.
   - **Move to Reference** — still useful, not a record of finished work.
   - **Trash** — genuinely spent.
3. Only after the user approves specific items, call `delete_note` on those. This moves
   them to trash and is recoverable via `restore_note`.
4. **Say clearly that emptying the trash is the user's job** in the Evernote app. Never
   describe your step as a permanent delete, and never offer to make it one.
5. **Audit the sleeping items** — `tag:"#tickler"` with no open task or reminder is a lost
   item that will never resurface. Give it a date or reclassify it. Same for
   `#someday_maybe` items that have sat untouched for a year: promote, tickler, or drop.
6. **Areas of Focus review (Horizon 2)** — surface the **Areas of Focus** note from
   Reference and read the areas back one at a time. For each, cross-reference the live
   `#gtd_project` list and ask directly:
   - *Does this area have an active project right now?*
   - *If not, is that a deliberate choice this quarter, or is it being quietly neglected?*

   An area with no project for a quarter is the signal this step exists to catch — usually
   health, finances, or home, because work always shouts louder. Do not propose projects
   unprompted; surface the gap, name it plainly, and let the user decide. Anything they
   commit to becomes an Inbox capture, clarified in the normal way.

   If no Areas of Focus note exists yet, say so and offer to draft one from the roles the
   user actually holds. Do not invent areas for them.
7. **Consolidate legacy tags** while here: `Needs Review` → `#review`, `Research` →
   `@research`, `12WY` → `#12WY`.
8. Report what was kept, moved, and trashed, and which areas came up short.

---

## First run only — the Actions sweep

Actions is currently doing the Inbox's job: live work notes sit alongside unclarified
personal captures. Before the first proper daily review, offer a one-time sweep.

1. List `notebook:"Actions"` in full.
2. Separate the genuinely live work items from the raw captures.
3. Run clarify + organise over the raw ones, in batches of about five, approving as you go.
   Personal items are in scope — they get `@home` and the same treatment.
4. Do not touch a note the user identifies as live work without asking.

Expect this to take one sitting. Afterwards, Actions holds only tagged, clarified items and
the Inbox becomes the real front door.
