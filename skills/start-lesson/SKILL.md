---
name: start-lesson
version: 1.0.0
description: >
  Creates a correctly-named, correctly-located notes file for a new lesson
  by copying template.md, so filenames and paths always match the phases/
  folder exactly (no typos, no drift). Resolves the lesson live from
  phases/, not a generated file.
  Trigger phrases: "start lesson", "starting the next lesson", "new lesson notes",
  "begin next lesson"
tags: [curriculum, progress, ai-engineering]
---

# Start Lesson

Scaffolds a new lesson notes file so it's found correctly by
`todays_agenda` and `update_dates` later. Never hand-type a filename —
always resolve it live from the `phases/` folder.

## Step 1 — Resolve which lesson

If the learner names a lesson, resolve it against `phases/`: run
`ls phases | sort` to find the phase, then `ls phases/<phase> | sort` to
find the lesson folder, confirming the title via
`phases/<phase>/<lesson>/docs/en.md`. Same match rules as `update_dates`
(exact → single fuzzy with confirmation → ask if ambiguous).

If they just say "start the next lesson," compute "next lesson" the same
way `todays_agenda` does (its Step 3): last notes file with `Date
completed` filled in, walked against the live-computed sequence from
`phases/`, plus one.

## Step 2 — Check it doesn't already exist

Expected path: `notes/<phase-folder>/<lesson-folder>.md`. If it already
exists, don't overwrite — tell the learner it's already there, and whether
`Date completed` is filled in or it's still in progress.

## Step 3 — Create it

Copy `notes/template.md` to the correct path (creating the phase
subfolder if this is the first lesson of a new phase). Fill in the
lesson's phase/lesson heading using the title read from `docs/en.md` —
don't leave placeholder text. Everything else stays blank.

## Step 4 — Confirm

State the file path created and the lesson title. One line, no filler.

## Edge cases

- **Phase folder in `phases/` has no corresponding lesson folders yet**
  (e.g. upstream added an empty placeholder phase): say so, don't create a
  notes file for a lesson that doesn't exist.
- **Ambiguous "next lesson" due to a gap** (an earlier lesson in sequence
  was skipped): flag it rather than silently picking one interpretation —
  same as `todays_agenda`'s gap handling.
