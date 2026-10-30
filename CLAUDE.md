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

The incident list, architecture principles and coding rules for this app live in the `workout-tracker` skill (section "Never repeat / principles / coding rules"). Read it before any edit.
