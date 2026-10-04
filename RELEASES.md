# Releases

Save points for the spearfishing workout tracker (`index.html`).  
To revert to any release: `git checkout <tag> -- index.html`  
To view what's in a release: `git show <tag>:index.html`

---

## Unreleased — Two-lift-day volume + protein (2026-10-03)

### What changed

- Plan (Supabase `david_plan`, both weeks): main compounds 2 → 3 sets (Floor Press, Landmine Press, Lat Pulldown, Goblet Squat, Cable Chest Press, Chest-Supported Row, Hip Thrust, Box Step-Up). Added Seated Leg Curl (Lift A) and Seated Calf Raise (Lift B). Accessories stay at 2 sets. Previous plan backed up at `~/.claude/doctor-backup/david_plan.2026-10-03.json`. Sessions already opened keep their old set counts.
- Progression suggestion is reps-only: never suggests a heavier load; at the top of the rep range it says to hold the load. A ↓ tap still suggests backing off 5 lb.
- Dive-eve notice on Lift A / Lift B when a Dive session is on tomorrow's calendar (`diveEveNotice()`): Lift A skips the leg lifts, Lift B is skipped.
- Nutrition tab: protein ~140g → ~160g (30–40g per meal); 1 whey scoop added to Meal 2, rice at Meal 4 cut to 1 cup.
- EXERCISE_INFO entries for Seated Leg Curl and Seated Calf Raise.
- YouTube search links on all 14 plan lifts; full EXERCISE_INFO entries for Face Pulls and Chest-Supported Row.
- Calendar day picker now matches the program: Lift A, Lift B, Apnea, Yoga, Dive, Rest, Fitness Test (Legs and Cardio removed).
- Removed the Week tab (and its code/CSS); the calendar now has Month and Nutrition tabs.
- Lift A / Lift B no longer append an apnea block (apnea is its own day). Older lift sessions with logged apnea data still show it.
- Hotel A / Hotel B (no equipment) in the day picker. Sessions carry `variant: 'hotel'`; templates live in the `HOTEL_PLAN` constant (not Supabase). Hotel A: feet-elevated push-up 3, pike push-up 3, table inverted row 3, rear-foot-elevated split squat 3, bridge walk-out 3, prone Y-T-W 2, hollow body 2. Hotel B: tempo push-up 3, backpack row 3, single-leg hip thrust 3, tempo reverse lunge 3, single-leg calf raise 3, wall sit 2, side plank 2. Same set totals (19) as the gym lifts; progression is slower tempo / more elevation / backpack load. `getLastSession(type, variant)` keeps hotel and gym history separate; `isTimeExercise` now detects "20–30s".

## Unreleased — Auto-rotating apnea tables (2026-10-03)

### What changed

- Apnea day now rotates **CO₂ → Dynamic walk → O₂** automatically, based on the last apnea day with a logged hold. Stored on the session as `apnea_table_mode` at creation; CO₂/Walk/O₂ buttons override. Next Up banner and apnea block title show today's table. `suggestApneaMode()`, `getApneaMode()`, `setApneaTableMode()`.
- Tables rescaled to tested maxes (static 2:50, walking 1:50), 8 rounds each: CO₂ 8 × 1:30 (rest 1:45 → 0:30); O₂ 1:00 → 2:10 (rest 2:00); new Dynamic walk 8 × 1:10 (~100 steps, rest 2:00 → 0:45) logging duration + steps.
- Previous apnea day was 6 × 0:50, far below ability. Push-day apnea unchanged.

---

## v1.5.0 — Freediving-first program overhaul (2026-10-02)

**Tag:** `v1.5.0`
**Commit:** fd5d493

### What changed

**Complete program and app overhaul driven by two findings:** pec/bicep tendon near-miss under TRT (muscle strength outpacing tendon remodeling), and overtraining suppressing the mammalian dive reflex (previous 8-day rotation caused sub-60s holds and inconsistent 50–60 ft depths; after 2-week break: 60–70 ft consistently, 75–81 ft multiple times).

**New 7-day rotation:** Lift A / Apnea / Yoga / Lift B / Rest / Apnea / Rest — down from 6 sessions/8 days to 2 lift sessions/week. `WEEKLY_SEQUENCE` constant drives the `getNextWorkoutType()` function.

**Two new day types — Yoga and Apnea standalone:**
- Yoga day: 11-pose checklist with tap-to-expand instructions, check buttons, and YouTube search links. `renderYogaSession()`, `toggleYogaPose()`.
- Apnea day: CO₂/O₂ mode toggle, 6-round tracking. `renderApneaStandaloneSession()`, `setApneaStandaloneMode()`, `handleApneaStandaloneInput()`.
- Calendar colors: `--yoga: #a855f7`, `--apnea: #0ea5e9`. Type labels, tile badges, type sheet buttons added.

**Next workout banner:** `#next-workout-banner` on calendar screen shows next session type/color with a tap-to-open button. `getNextWorkoutType()` reads last 14 days of sessions to find position in WEEKLY_SEQUENCE; `renderNextWorkoutBanner()` populates it; `openNextWorkout()` creates and opens the session.

**Nutrition tab:** 3rd tab in calendar `cal-tabs` alongside Month/Week. `renderNutritionView()` shows macros, 4 meals with quantities, collagen protocol, supplements, dive-day nutrition rules, tendon protocol.

**Video links:** `EXERCISE_INFO` and `YOGA_POSES` entries include `video:` YouTube search URLs. Exercise info modal now renders a video link row when present. Yoga pose cards show YouTube links inline.

**Tendon-safe exercise selection (permanent exclusions):**
- Removed: flat barbell bench press, weighted dips (pec tendon), incline DB curl, supinated heavy curls (bicep tendon)
- New Lift A: Floor Press, Landmine Press, Neutral-Grip Lat Pulldown, Goblet Squat, Face Pulls, Hollow Body Hold
- New Lift B: Cable Chest Press, Chest-Supported Row, Hip Thrust, Box Step-Up, Cable Pull-Through, Pallof Press

**WARMUP_INFO** updated for Lift A/B with isometric tendon prep; added yoga and apnea entries.

**Apnea targets recalibrated to 65% of current demonstrated max** (~50s holds) — previous 1:45 targets were above-max, counterproductive. `APNEA_TARGETS` gains `apnea_co2` and `apnea_o2` entries for standalone sessions.

**Nutrition reduced from 3,170 → 2,800 kcal/day, protein from ~194g → ~140g.** Excess protein raises CO₂/calorie, directly shortening breath holds.

---

## v1.4.0 — Time-mode set rows for timed exercises (2026-08-26)

**Tag:** `v1.4.0`
**Commit:** 619fa4f

### What changed

**Timed exercise set rows.** Exercises with `timed: true` in `EXERCISE_INFO` (e.g., Hollow Body Hold) render a duration input instead of weight × reps. Time stored as seconds. Set row shows M:SS format. `isRealSet` checks `duration > 0` for timed sets. `plan_reps` hint displayed in the card target line as usual.

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
