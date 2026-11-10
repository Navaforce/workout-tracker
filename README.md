# Spearfishing Tracker

Personal workout tracker built around spearfishing and freediving: strength lifts, apnea training and yoga on a repeating cycle, with a dive logged as an apnea day.

**Live app:** https://navaforce.github.io/workout-tracker/

Single HTML file (`index.html`), no build step, no CDN dependencies. Mobile only (designed for a 430 px wide screen) and installable to the iPhone Home Screen.

## Features

- **Calendar** of sessions by day type, with a day picker and a cycle summary (tap the fish icon).
- **Three-lift cycle:** Lift A, Yoga, Lift B, Apnea, Lift C, Apnea + Yoga, Rest. Pre-Dive variants of each lift keep to upper-body work, and no-gym Hotel variants need no equipment.
- **Diving aware:** a dive counts as apnea, so the next day becomes rest or a lift automatically, and "Next Up" suggests the Pre-Dive lift when a dive is planned for tomorrow.
- **Exercise cards** with set rows, prefilled weights and cues, and apnea blocks.
- **Yoga timers** per pose, ending with Uddiyana Bandha holds.
- **Cloud sync:** workout plans and logs sync through a Supabase project; data is also cached in localStorage.

## Files

| Path | Purpose |
|---|---|
| `index.html` | The whole app |
| `RELEASES.md` | Release history and how to revert to a tagged version |
| `CLAUDE.md` | Working notes for AI-assisted editing |
| `apple-touch-icon.png` | Home Screen icon |

## Reverting

Releases are tagged. To restore a version: `git checkout <tag> -- index.html`. To inspect one: `git show <tag>:index.html`.

## Deploy

Push to `main`; GitHub Pages serves the root.
