---
name: update-dates
version: 1.0.0
description: >
  Marks a lesson-completion or review field as done in a lesson notes file,
  filling in today's date and any fumbled-on notes, based on an explicit
  instruction from the learner. Never infers what happened from conversation
  alone — only acts on explicit instructions. Resolves lesson names live
  against the phases/ folder, not a generated file.
  Trigger phrases: "mark cold review done for", "mark weekly review done for",
  "mark phase-end review done for", "mark lesson complete", "update dates",
  "I finished the review for"
tags: [curriculum, progress, review, ai-engineering]
---

# Update Dates

You update fields in a lesson notes file under `notes/<phase>/<lesson>.md`,
acting **only** on an explicit instruction naming the lesson and which
field to update — e.g.:

> "Mark cold review done for Terminal & Shell, fumbled on tmux detach"
> "Mark weekly review done for Terminal & Shell and Linux for AI, no fumbles"
> "Mark Terminal & Shell as complete"

Do not infer from earlier chat that a review happened unless the learner
says so in this instruction. If the field to update isn't specified, ask.

## Step 1 — Resolve the lesson

Try, in order, stopping at the first success:

1. **Check existing notes files first.** Most instructions target a lesson
   that already has a file (in progress or done). Search `notes/` for a
   filename or in-file heading matching the given name.
2. **If no notes file matches, resolve live from `phases/`.** Run
   `ls phases | sort` and `ls phases/<phase> | sort` as needed, and check
   lesson titles via `phases/<phase>/<lesson>/docs/en.md` for a title
   match. This covers lessons just started that don't have a file yet —
   though `start_lesson` should normally create the file first.

- **Exact match:** use it directly.
- **Single fuzzy/partial match:** confirm back to the learner what you
  resolved it to (e.g. "Updating Phase 00 / Lesson 10 — Terminal & Shell")
  rather than silently proceeding.
- **Multiple matches:** list candidates, ask which one.
- **Resolved lesson has no notes file yet:** don't create one here — tell
  the learner to run `start_lesson` first.

## Step 2 — Resolve the field

| Instruction says | Field to update |
|---|---|
| "complete", "done with the lesson" | `Date completed:` |
| "cold review", "next-lesson review" | `Next-lesson cold review` |
| "weekly review", "+1 week" | `+1 week review` |
| "phase-end review", "phase review" | `Phase-end review` |

Multiple lessons named for the same field in one instruction (batch weekly
review) → update each file individually.

## Step 3 — Check for overwrite

If the target field already has a date filled in, don't silently overwrite
— show the existing value and ask for confirmation, unless the learner's
instruction explicitly says to correct/redo it.

## Step 4 — Write the update

The learner's actual template has no checkbox — a review block looks like:

```
**Next-lesson cold review** (exercises, no notes):
done date: ___
fumbled on:
-
-
```

There is no ⬜/✅ marker anywhere. The presence of an actual date in
`done date:` is what marks it done — do not invent a checkbox.

For `Date completed:`, just fill in today's date after the colon.

For a review block (`Next-lesson cold review` / `+1 week review` /
`Phase-end review`):
- Replace the `___` after `done date:` with today's actual system date.
- Under `fumbled on:`, replace the first blank `-` bullet with what the
  learner said, verbatim. If they said there were no fumbles, write
  `- none`. Leave any remaining blank `-` bullets as-is (the learner may
  add more later). Never invent a fumble that wasn't stated, and never
  leave `done date: ___` unfilled once you're marking the review done — if
  the learner didn't say what was fumbled and didn't say "none" either,
  ask before writing anything.

Use exact string replacement so the rest of the file — cheat-sheet,
gotchas, why-it-matters — stays untouched. Read the file first to confirm
its exact current structure before replacing, since template formats can
still evolve.

## Step 5 — Confirm

State plainly what changed: file path, field, date, fumbled-on. One or two
lines, no more.

## Edge cases

- **Lesson referenced by number only** ("mark lesson 10 done"): resolve
  within the phase currently in progress unless a phase is also named; ask
  if still ambiguous.
- **Vague instruction** ("update the dates", no lesson/field named): don't
  act, ask what to update.
- **Upstream repo changed a lesson's folder name/number since the notes
  file was created:** the notes filename won't match the new `phases/`
  slug. Flag this mismatch to the learner rather than silently treating it
  as a different lesson — don't auto-rename their notes file.
