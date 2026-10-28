# Claude Instructions

## Preferences

- **Responses**: Terse and direct. No trailing summaries ("I've done X" recaps). No emojis unless asked.
- **Comments in code**: Default to none. Only add a comment when the WHY is non-obvious.
- **Context window**: Statusline configured at `~/.claude/statusline-command.sh` — shows a draining bar + % remaining + tokens left (e.g. `████░░░░ 62% remaining (124k / 200k tokens left)`). Colors: green >50%, yellow >25%, magenta >10%, red ≤10%. Wired into `~/.claude/settings.json`. Do not reconfigure unless asked.
- **Tech debt**: If the user suggests something that could introduce tech debt, caution them, explain why it's bad, and recommend the correct approach instead. Never silently implement a bad pattern.
- **Docs folder for every project**: every project gets a `docs/` folder with a `BRIEF.md` at creation (sf-demo-template already includes one; for any other project create it before the first commit).
- **Keep docs and skills current**: After any build that changes a pattern, gotcha, process, URL/config fact, or reverses an earlier assumption, update the relevant skill(s), project `docs/*.md`, and ledger/project memory in the same turn, without being asked. List the files changed in one line. Don't wait for the commit request.

---

## Projects

### Workout Tracker — `/Users/dnava/index.html`

Live app. Single-file HTML, no build step, no CDN dependencies. Deployed via GitHub Pages. Mobile-only (430px assumed). Backup of all historical workout data at `/Users/dnava/wt_data_backup.json`. jsdom available at `/tmp/wt-test/node_modules/jsdom` for testing.

Reference sections (schema, Supabase, UI, calendar, fitness, exercise card, modals, git/releases) live in the `workout-tracker` skill. Load it before editing `/Users/dnava/index.html`.

#### What went wrong in the old app — never repeat these

1. **Day state on plan objects** — `date`, `completed`, `startedAt`, `completedAt` stored on `plan.days[i]`. Plan upload overwrote them, causing calendar date corruption.
2. **Set-count sync** — picked the side with more sets. Failed on pre-fills, deletions, cross-week logging. Caused infinite push loops.
3. **Repair in sync path** — `_repairWeekData` cleared session state → `cloudRepaired=true` → pushed local back to cloud → infinite loop every 3 seconds.
4. **`fillFromLastWeek` timestamps** — all copied sets got `ts: Date.now()` at the same millisecond, making fill copies indistinguishable from real logged sets.
5. **Fuzzy name matching** — caused ghost matches and wrong pre-fills across weeks.
6. **`save()` calls `scheduleSync()`** — any write inside the sync path re-triggered sync.
7. **`showScreen()` id mismatch** — `showScreen('calendar')` looked for `id="calendar-screen"` but element is `id="cal-screen"`. Back button silently failed. Fix: map `'calendar'` → `'cal-screen'` explicitly.
8. **`.onclick` property assignment** — browser ignores JS `.onclick =` when an HTML `onclick=""` attribute exists. Always use HTML `onclick="funcName()"` attributes. Never assign `.onclick` at runtime.
9. **`confirmModalOk` callback cleared before called** — `closeConfirmModal()` set `window._confirmCallback = null` before the callback fired. Fix: capture `cb = window._confirmCallback` before calling `closeConfirmModal()`.
10. **BW mode `prefilled` deadlock** — `handleRepInput` only cleared `prefilled` inside `if (isRealSet(set))`, but `isRealSet` requires `!prefilled`. In BW mode nothing else cleared it (no `handleWeightInput` call), so reps were stored but set stayed invisible. Fix: clear `prefilled` when `reps > 0` unconditionally, before the `isRealSet` check. Also clear it in `handleBWInput`.
11. **`handleNoteInput` never updated DOM** — saving a note updated localStorage but never toggled `has-note` on the ⓘ button. Set 0 never got the cyan indicator because nothing else (effort/adjustment pill taps) happened to re-render it. Fix: `handleNoteInput` now directly updates the `.note-toggle` button's `has-note` class via `btn.classList.toggle('has-note', !!(val && val.trim()))` after saving.

#### Core architecture principles

1. **Sessions are first-class records** — ID: `david_YYYY-MM-DD_type`. Created when a day is opened. Never touched by plan upload.
2. **Plan is read-only template** — `session_id = 'david_plan'` in Supabase. Never touches session records.
3. **Sync by `updated_at` timestamp, per session** — last-write-wins. No set-count comparisons. `save()` never called from inside `syncNow()`.
4. **`isRealSet(s)` = `Number(s.reps) > 0 && !s.prefilled`** — the only gate for "did the user log this." Explicit `Number()` coercion — reps may be stored as strings from input events.
5. **`session.date` written once** — on first real rep entry. Never overwritten.
6. **Exact exercise name matching only** — no fuzzy matching. Skill enforces consistent names.
7. **Test before implement** — write a Node.js test reproducing the failure, run it, confirm passing, then implement. Use jsdom for DOM tests: `require('/tmp/wt-test/node_modules/jsdom')`.
8. **`window._confirmCallback`** — generic confirm modal callback. Always capture to local var before `closeConfirmModal()`.

#### Coding rules

- **Read before write** — always read the file before editing
- **Test before deploy** — jsdom tests for any navigation, data, or UI logic. Use `url: 'http://localhost/'` in JSDOM constructor (required for localStorage).
- **No grey on readable text** — `--text` for all content, `--muted` for decorative only
- **`CSS.escape` not available** — use `sessionId.replace(/[^a-zA-Z0-9_-]/g, '\\$&')` for DOM queries
- **Sync safety** — `syncNow()` writes via `localStorage.setItem()` directly, never `save()`
- **No `.onclick` property assignment** — always HTML `onclick="..."` attributes
- **Python for multi-line template literal edits** — Edit tool fails on backtick strings spanning lines; use `python3 -c "..."` with string replacement instead
- **`seedFitnessHistory()` must not bump `_meta_updated_at`** — see Supabase meta record note above
- **`getLastSession(type)` excludes current session and sessions with no real sets** — uses `currentSessionId` global. `getLastExerciseSets(exName, dayType)` searches per-exercise across all past sessions for the most recent real sets for a given exercise name.
- **Supabase calls only via `sbFetch()`** — never raw `fetch` to the REST API (auth token + 401 handling live there). Sign-in is built but server RLS is not enabled yet.
- **`markSessionDeleted` must not bump `_meta_updated_at`** — a stale device would overwrite fitness tests/injured days.
- **One session per date; hotel sessions have `_hotel` id suffix** — look sessions up with `findSessionForDate`, never `Object.entries(...).find(s.date===iso)`.
- **Escape all interpolated data with `esc()`** (escapes `'` too); no cloud text inside JS strings in `onclick`.
