# Releases

Save points for the spearfishing workout tracker (`index.html`).  
To revert to any release: `git checkout <tag> -- index.html`  
To view what's in a release: `git show <tag>:index.html`

---

## v1.3.3 — Ghost session fix (2026-06-30)

**Tag:** `v1.3.3`

### What changed

**Type sheet opening on days with existing data after "Remove Day" + retype.** Three root causes fixed:

1. **`supabaseDelete` added.** `deleteRestDay` now calls `supabaseDelete(sid)` before `syncNow`. Previously, the sync immediately re-fetched the just-deleted session from Supabase and restored it locally — every tap saw the zombie session (no real sets) and re-opened the type sheet.

2. **`findSessionForDate` made data-aware.** When multiple sessions exist for the same date (orphan from a previous type + current session), the function now prefers the session with real data (`dive_log` fields, real sets, or `completed`) over an empty one. Falls back to first-found if none have data.

3. **Dive `hasData` check in `calDayTap`.** Dive sessions have no `exercises` array — the old check always evaluated false, so dive tiles always opened the type sheet. Now uses `dive_log` fields (dives, max_depth_ft, max_hold_secs, note) as the data presence test.

**Supabase cleanup:** Two orphan rows deleted directly (`david_2026-06-26_dive`, `david_2026-06-28_pull`).

---

## v1.3.2 — Dive day type (2026-05-30)

**Tag:** `v1.3.2`

### What changed

**Dive added to the day-type picker.** Selecting Dive on any calendar tile opens a dedicated log screen with three fields: Dives (count), Max Depth (ft), and Max Breath Hold (M:SS), plus a notes field. No exercise cards, no apnea block. Calendar tiles show dives count and depth. Syncs to Supabase with the rest of the session. Stored in `session.dive_log`.

---

## v1.3.1 — Apnea round effort rating (2026-05-30)

**Tag:** `v1.3.1`

### What changed

**Effort pills on apnea rounds.** Each breath-hold round now has Easy / Mod / Hard pills below the input fields, styled identically to the exercise set effort pills. Tapping a pill selects it; tapping the active pill deselects it. Stored in `session.apnea.r{n}.effort` and synced to Supabase with the rest of the session on the next sync.

---

## v1.3.0 — Exercise drag-to-reorder (2026-05-30)

**Tag:** `v1.3.0`

### What changed

**Drag-to-reorder exercises within a workout day.** Long-press the ⠿ grip handle on the left of any exercise header and drag up or down to reposition it. A ghost card follows the finger; the target card gets a cyan drop-indicator bar. On release the exercise array is spliced to the new position, saved, and the screen re-renders. Open/collapsed card state is remapped to new indices so nothing snaps shut. The tap-to-collapse action on the header is suppressed after a drag so releasing your finger never accidentally toggles the card.

---

## v1.2.2 — Rest day label + delete (2026-05-26)

**Tag:** `v1.2.2`

### What changed

**Rest label missing from month view.** Month view excluded rest days from the type label by checking `!isRest` — removed that guard so rest days render `Rest` the same way all other types do.

**Rest day tap opened workout screen (blank).** `calDayTap` sent rest sessions to `openWorkout`, which renders an empty screen. Fixed to route rest day taps to the type sheet instead.

**No way to delete / reassign a rest day.** Added `deleteRestDay(iso)` and a "Remove Day" button (red, bottom of type sheet) that appears only when the tapped day has a rest session. Tap → removes session from storage, syncs, re-renders. Day returns to empty state, type sheet can be reopened to re-pick.

---

## v1.2.1 — BW mode reps bug fix (2026-05-22)

**Tag:** `v1.2.1`

### What changed

**Calendar not re-rendering after back navigation.** `goBack()` called `showScreen('calendar')` but never `renderCalendar()`, so tile states (completed, partial, etc.) were stale until the next sync. Fixed by adding `renderCalendar()` to `goBack()`.

**BW mode reps never counted as real sets.** In bodyweight mode, sets are created with `prefilled: true`. The normal flow clears `prefilled` inside `handleRepInput` → `if (isRealSet(set))` — but `isRealSet` requires `!prefilled`, making it a deadlock: prefilled can never clear itself. In non-BW mode `handleWeightInput` breaks the deadlock by clearing `prefilled` when weight is typed; BW mode has no weight typing (value is pre-populated). Fix: clear `prefilled` directly when `reps > 0`, before the `isRealSet` check. Also cleared `prefilled` in `handleBWInput` for the case where user touches the BW input first.

---

## v1.2.0 — P3 performance + minor quality (2026-05-21)

**Tag:** `v1.2.0`

### What changed

**P3#11 — `getLastSession` called once per render, not once per exercise card.** `renderExercises` now fetches `lastSession` once and passes it into `renderExerciseCard`. Added `computeProgressionFromSession(exName, lastSession)` to avoid the redundant call inside `computeProgression`. For a 6-exercise session this cuts `load()`+`JSON.parse()` from ~7 calls to 2.

**P3#12 — `computeProgression` uses last set's weight instead of first.** Changed `realSets[0].weight` to `realSets[realSets.length - 1].weight`. The final set is a better indicator of where to start next session (accounts for warmup progressions like 175→180→185 lb).

**P3#13 — Removed dead `calYear !== undefined` / `calMonth !== undefined` guards.** `init()` always sets both before any render call, so these checks never evaluated the fallback branch.

---

## v1.1.0 — P2 dead code + latent bugs (2026-05-21)

**Tag:** `v1.1.0`

### What changed

**P2#6 — Removed `_origSaveDurModal` dead code.** The `const _origSaveDurModal = ...` line always captured `null` (defined before `saveDurModal`) and was never read.

**P2#7 — Removed swipe-to-delete dead code.** `swipeStart`, `swipeMove`, `swipeEnd`, `swipeStartX`, and `.set-row.swiped` CSS were never attached to any element. CLAUDE.md documents no swipe-to-delete.

**P2#8 — Removed orphaned `.is-pr` class emission.** The CSS rule `.ft-metric-val.is-pr` was removed in a prior session (all values are cyan); the class was still being added to the DOM. Also removed the `isPR` IIFE that computed it.

**P2#9 — Fixed `getFitnessBest`, `ftTrendIcon`, and `prCount` BSS comparison.** All three were comparing BSS `{weight, reps}` by `.reps` only — 20 reps at 25 lb incorrectly beat 18 reps at 50 lb. Added `bssScore(v) = v.weight * 1000 + v.reps` composite. Weight takes precedence; reps break ties.

**P2#10 — Explicit `Number()` coercion in `isRealSet`.** Changed `s.reps > 0` to `Number(s.reps) > 0` to make the string-to-number coercion explicit and safe.

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
