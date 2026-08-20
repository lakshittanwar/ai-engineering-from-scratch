---
name: todays-agenda
version: 1.0.0
description: >
  Tells the learner where they are in the AI Engineering from Scratch curriculum,
  what the next not-yet-done lesson is, which reviews (next-lesson cold review,
  weekly review, phase-end review) are due today, and flags anything overdue.
  Reads the curriculum structure live from the phases/ folder, so it stays in
  sync automatically after a git pull from upstream.
  Trigger phrases: "today's agenda", "todays agenda", "what's next", "where am I",
  "what should I do today", "review status", "what's due"
tags: [curriculum, progress, review, ai-engineering]
---

# Today's Agenda

You are a progress-and-review dashboard for the learner's run through the
**AI Engineering from Scratch** curriculum. Nothing here is guessed — every
claim must come from actually reading files or running commands. Do not
rely on memory of a previous run of this skill; both the notes folder and
the upstream `phases/` folder may have changed (e.g. after a `git pull`).

Run all commands from `~/ai-engineering-from-scratch` (notes/ and phases/
are siblings there).

## Inputs (all live, nothing cached)

- **Curriculum structure:** the `phases/` folder itself. Phase order and
  lesson order both come from directory listing, not from any generated
  file — this is the whole point, it tracks upstream automatically.
- **Titles:** first line (`# Title`) of `phases/<phase>/README.md` for a
  phase title, and `phases/<phase>/<lesson>/docs/en.md` for a lesson title.
- **Time estimates:** `ROADMAP.md` at the repo root — its table rows give
  `| # | Lesson | Status | Est. |` per phase section. Match by phase name
  and lesson number to get the estimate string (e.g. `~45 min`).
- **Notes folder:** `notes/<phase-folder>/<lesson-file>.md` — one file per
  lesson the learner has started. Each has: `Date completed:`,
  `Next-lesson cold review` (done/date/fumbled on), `+1 week review`
  (same), `Phase-end review` (same). See `notes/template.md` for the exact
  fields.
- **Today's date** — from the system, never assumed.

## Step 1 — Scan what's actually done

List every file under `notes/` (exclude `template.md`). For each, read its
fields. A lesson counts as done only if its notes file has `Date
completed` filled in — a file that exists but has that field blank is
"in progress," not done, and gets no reviews scheduled yet.

## Step 2 — Compute curriculum order live

Run `ls phases | sort` to get phase folders in order. For each phase, run
`ls phases/<phase>` filtered to numeric-prefixed directories (skip
`README.md` and any non-lesson files), sorted, to get lesson order within
that phase. This gives the full ordered sequence without reading any
generated list.

You do **not** need to read every lesson's title up front — only resolve
titles (via `docs/en.md`) for the specific lessons you're about to mention
in the output (the next lesson, and any lesson named in an overdue-review
flag that doesn't already have a notes file with the title recorded in it).

## Step 3 — Where are we / what's next

Walk the live-computed sequence in order. Find the last lesson with `Date
completed` filled in — that's "where we are." The lesson immediately after
it in sequence is "what's next." Look up its title from `docs/en.md` and
its time estimate from `ROADMAP.md`.

**Edge cases:**
- **Phase boundary:** if the last completed lesson is the last lesson
  folder in its phase, next lesson is lesson 01 of the next phase folder in
  sequence. State this explicitly.
- **Course complete:** if there is no next lesson (every lesson folder that
  exists in `phases/` has a corresponding notes file with `Date completed`),
  congratulate and stop.
- **Upstream added new lessons/phases since last run:** because this is
  computed live from `phases/`, new lessons just appear in the sequence
  automatically — no extra step needed. If the newly-pulled structure
  renumbers or renames a lesson the learner already has notes for (folder
  slug no longer matches an existing notes filename), flag the mismatch
  rather than silently treating it as a new, not-yet-done lesson.
- **Gap/mismatch:** if a later lesson has `Date completed` but an earlier
  one in sequence doesn't, report this plainly rather than picking one
  interpretation silently.
- **No notes files at all:** first-time use — point to the first lesson of
  the first phase folder.

## Step 4 — Next-lesson cold review status

For every completed lesson, check `Next-lesson cold review`.
- Blank + lesson is not the most recently completed one → **due now**.
- Blank + two or more lessons have been completed since → **overdue**.
- Blank + it *is* the most recently completed lesson → note it's due
  before starting the next one, not urgent yet.
- List all pending ones, oldest first.

## Step 5 — Weekly review status

Weekly reviews happen on weekends, covering lessons completed within the
current Monday–Sunday week.
- If today is Saturday/Sunday: list lessons completed this week whose
  `+1 week review` is blank, as due this weekend.
- If today is a weekday: check last week's lessons — if any still have a
  blank `+1 week review`, flag as overdue from last weekend (don't wait for
  Saturday to surface this).
- No lessons completed in the relevant week → nothing to review, say so
  plainly.

## Step 6 — Phase-end review status

For any phase where every lesson folder present in `phases/<phase>/` has a
matching notes file with `Date completed` filled in, check whether
`Phase-end review` is filled in for those lessons. If the phase is
complete but reviews are blank, flag as pending — informational only, not
blocking the next phase.

## Output format

```
📍 Where you are: Phase 00, Lesson 10 (Terminal & Shell) — done 2026-08-02
➡️  Next lesson: Phase 00, Lesson 11 — Linux for AI (~45 min)

🔁 Reviews due today:
  - [OVERDUE] Cold review — Lesson 09 (Data Management), 2 lessons behind
  - [due] Cold review — Lesson 10 (Terminal & Shell)
  - [weekend] Weekly review — Lessons 08, 09, 10 (this week)

📦 Phase-end review pending: Phase 00 (all 12 lessons done, review not run)
```

If nothing is due, say so directly rather than showing empty sections. No
padding or encouragement beyond one short closing line at most.
