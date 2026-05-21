# Releases

Save points for the spearfishing workout tracker (`index.html`).  
To revert to any release: `git checkout <tag> -- index.html`  
To view what's in a release: `git show <tag>:index.html`

---

## v1.0.0 — Fitness sync + 5-bug fix (2026-05-21)

**Tag:** `v1.0.0`  
**Commit:** see tag

### What's in this release

**Features complete as of this save point:**

- Full workout tracker — Push / Pull / Legs / Cardio / Rest / 8-day rotation
- Calendar screen (month + week strip views) with type-colored tiles, set counts, duration, progress bars
- Workout screen — exercise cards, set rows (weight/reps/effort/adjustment/notes), apnea block (6 rounds, per-day-type targets), Mark Complete validation
- Timer panel — stopwatch + countdown tabs; preset buttons (hold-to-delete); pip overlays for both timers when panel is closed
- Fitness test screen — 10 built-in fields (including compound BSS and walking-apnea-with-steps), dynamic field config (add/remove/reorder), prev/best display, tooltips, notes per field, duration tracking, auto-save on input, history with PR badges, delete records
- Supabase sync — last-write-wins per session; plan record (`david_plan`); meta record (`david_meta`) for fitness_records, injured_days, fitness_test_days
- Header — Spearo + hogfish SVG, dumbbell icon (fitness), cloud icon (sync), circular-arrow (reload), warmup ⓘ, timer ⏱
- iOS standalone PWA — `hardReload()` uses `location.reload(true)`

**Bugs fixed in this release (from review):**

1. Fitness records, injured days, and fitness-test days were never synced to Supabase — now stored in `david_meta` record
2. `prevNote` placeholder dropped after pill tap (`refreshSetRow`) or set delete — fixed via `getPrevNotesForExercise()` helper
3. `getCurrentWeekNumber()` hardcoded to 5 — now returns max week number from loaded plan
4. Tapping a past fitness-test calendar tile reset the form date to today — `showFitness(iso)` now accepts optional date
5. Custom fitness field labels showed blank in history — now resolved via `cfgLabelMap` from live config

---

## How to create a new save point

After a significant change is working and tested, run:

```bash
# 1. Stage and commit
git add index.html RELEASES.md
git commit -m "vX.Y.Z — <short description>"

# 2. Tag it
git tag vX.Y.Z

# 3. Push commit + tag to GitHub
git push origin main
git push origin vX.Y.Z
```

Then add an entry to this file above (newest at top).

---

## Revert instructions

```bash
# See what changed between two releases
git diff v1.0.0 v1.1.0 -- index.html

# Revert index.html to a specific release (leaves RELEASES.md untouched)
git checkout v1.0.0 -- index.html

# After reverting, push the rollback as a new commit
git add index.html
git commit -m "revert: roll back to v1.0.0"
git push origin main
```
