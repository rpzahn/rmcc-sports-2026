# RMCC 2026 Sports Scheduler — Progress

## What we built

A single-file iPad web app for the RMCC 2026 camp week. It runs three concurrent 12-team double-elimination tournaments (Volleyball, Hockey, Softball) across the 34 thirty-minute sport slots, a standalone 12-team single-elimination Rafting bracket, and a camp-wide Standings view that combines all four brackets with seven non-bracket events.

### Files
- `index.html` — the entire app (~2790 lines). Vanilla HTML/CSS/JS, no build step.
- `.gitignore` — excludes the schedule PDF from the git repo.
- `2026 RMCC Daily Schedule.pdf` — local reference only, never committed.
- `.claude/launch.json` — config for a static preview server (`python3 -m http.server 8000`).

### Decisions locked in
- **Teams**: generic Team 1–12 (no name customization).
- **Gym-sport format**: double-elimination, **no bracket reset**, 22 bracket matches + 4 placement matches = 26 real matches per sport.
- **Gym-sport seeding (constrained)**: each team gets exactly one Round-1 bye across the three sports; the 12 first-round pairings are all distinct; round-2 rematch potential is minimized best-effort. Falls back to relax-and-warn if the unique-pairings constraint can't be met.
- **Schedule (34 slots)**: Sun 12 / Mon 9 / Tue 2 / Wed 5 / Thu 0 / Fri 6.
- **Placement (gym sports)**: every team gets a final place 1–12. Losers of LB R1/R2/R3/R4 play placement games for 11/12, 9/10, 7/8, 5/6 instead of just being eliminated.
- **Conflicts**: when a real match can't run (team excluded or playing another sport), defer it and substitute a filler/exhibition game (winner recorded for display only).
- **Filler balance**: the filler picker ranks candidates by fewest games already scheduled *in that sport*, to even out per-sport game counts. Eliminated teams are still preferred — their game count is final, so filling them in won't overshoot.
- **Grand Finals**: all three GFs fall in Friday slots, in three different Friday slots.
- **Rafting**: standalone 12-team single-elimination bracket — a main bracket, a 3rd/4th place game, and a separate 5th/6th place consolation bracket fed by the losers of the first two main rounds, per the camp's printed bracket. Not slot-scheduled — winners are tapped directly in the Brackets tab. Decides places 1–6; the six consolation non-finalists share places 7/9 (cosmetic — places 7–12 score 0 points). Randomly seeded; re-seedable from Settings.
- **Standings points**: editable per-place points table, defaults `[20, 17, 14, 11, 8, 5, 0, 0, 0, 0, 0, 0]` (places 7–12 score 0), applied to every bracket and event. Ties give each tied team the same points and skip the next place.
- **Non-bracket events**: seven pre-seeded — Mixers and Team Names (three 1–12 rankings summed, lowest total wins), Running Relay / Climbing Wall / Swimming Relay (enter times, fastest wins), Skits (three judges' scores summed, highest total wins), Kitchen Help (manual 1–12 placement).
- **Persistence**: `localStorage` on a single iPad.

## What's done and verified

| Feature | Status |
|---|---|
| 34-slot schedule definition (Sun–Fri blocks) | ✅ |
| 12-team double-elim bracket generator (gym sports) | ✅ |
| Placement games for clean 1st–12th (gym sports) | ✅ |
| Constrained seeding: one R1 bye per team, unique R1 pairings | ✅ |
| Bracket progression (winner advances; loser drops to LB or PL match) | ✅ |
| Per-team `place` recorded for all 12 teams when a bracket completes | ✅ |
| Per-slot scheduler with sport-order rotation for fairness | ✅ |
| No-double-booking rule across the 3 sports per slot | ✅ |
| Per-slot team exclusion (tap to toggle) | ✅ |
| Filler-game substitution, balanced by per-sport game count | ✅ |
| Grand Finals gated to Friday + forced into three separate Friday slots | ✅ |
| Tap-to-pick winner with auto-advance (gym sports, in Slot view) | ✅ |
| Clear/correct winner with downstream invalidation (incl. places) | ✅ |
| Manual team swap (downgrades real match to filler) | ✅ |
| Slot view with sport-color icons + sport cards | ✅ |
| Gym brackets view (per-sport WB/LB/PL/GF visualization with champion banner) | ✅ |
| SVG connector lines between matches within each bracket section | ✅ |
| **Rafting**: 12-team single-elim bracket — main + 3rd/4th + 5th/6th consolation | ✅ |
| Rafting bracket is interactive: tap winner, auto-advance, Clear, downstream invalidation | ✅ |
| Rafting tab in the Brackets view; randomly seeded; re-seedable | ✅ |
| **Standings tab**: overall ranking across 4 brackets + 7 events, live refresh | ✅ |
| Editable points-by-place (1st–12th), default 20/17/14/11/8/5/0… | ✅ |
| Per-team game-count grid (per gym sport + total) | ✅ |
| Placement events: manual 1–12 entry | ✅ |
| Time events: enter a time, fastest places 1st; colon auto-inserts during entry | ✅ |
| Judged events: three judges' scores summed, highest total places 1st | ✅ |
| Ranked events: three 1–12 rankings summed, lowest total places 1st | ✅ |
| Add/edit/delete custom events; four-way type chooser for new events | ✅ |
| Settings (re-seed all four brackets, export JSON, import JSON, full reset) | ✅ |
| localStorage persistence across reloads | ✅ |
| First-run "Generate Brackets" flow | ✅ |
| iPad-tuned CSS (large tap targets, system fonts, viewport meta) | ✅ |

### Verification done
- `jsc` syntax-checks the full script block — no parse errors.
- Browser preview (static server): full Rafting bracket playthrough — all 19 matches route correctly, all 12 teams placed (1–6 unique, 7×2, 9×4), champion set; re-deciding a result correctly invalidates downstream and re-routes the loser into the consolation bracket; the interactive tap / Clear flow works in the live UI.
- Ranked-event scoring (lowest sum wins, ties handled), time parsing, and the new points scale verified.
- Auto-colon time entry verified: typing `2 2 3 . 4` yields `2:23.4`, saved as 143.4 s.
- Settings re-seed regenerates all four brackets including Rafting.
- Earlier in development: full-week bracket simulation, heavy-exclusion edge cases, swap and persistence checks.

### Notes on persistence
- The `localStorage` key is `rmcc2026-sports-v2`. It was bumped from `v1` when the event list, points scale, and Rafting bracket landed — old `v1` saves are abandoned, so the app re-seeds fresh on the first load after the update.
- `ensureStateFields` backfills newer fields on load: the points table, an event `type` plus its data field (`places` / `times` / `scores` / `ranks`), and per-tournament `places`.

## What's left to do

### Required before camp
- **Real iPad smoke test**: open the file in actual iPad Safari, Add to Home Screen, exercise the flow in both portrait and landscape. The event editors (placement / time / judged / ranked) and the interactive Rafting bracket have been exercised in a desktop browser preview but not yet on a physical iPad. (Smoke test so far reported as looking good.)
- **Hosting**: GitHub Pages — repo at `github.com/rpzahn/rmcc-sports-2026`. Once Pages is enabled (Settings → Pages → deploy from branch `main`, `/` root), each push to `main` redeploys automatically; live URL `https://rpzahn.github.io/rmcc-sports-2026/`. Deployment requires a manual `git commit` + `git push`.

### Nice-to-haves (not blocking)
- **Recent winners log**: rolling "last 5 results" feed on the slot view.
- **Day-level exclusion**: "Team X out all of Tuesday" shortcut (currently per-slot only).
- **Sport-order rotation visibility**: surface which sport picks first in a contested slot.
- **Filler game opt-out**: setting to skip filler games entirely.
- **Print/screenshot view**: a clean printable bracket page for posting at camp.
- **PWA manifest + icons**: proper Add-to-Home-Screen icon and splash screen.
- **Cross-section bracket lines**: connectors stay within each bracket section; WB→LB, finals→GF, LB→PL, and the Rafting main→consolation links are not drawn.

### Known small UX gaps
- No undo for "Full reset" or "Re-seed all brackets" beyond the confirmation modal — JSON export is the only recovery path.
- The brackets view scrolls horizontally on narrow screens (intentional — 6 LB rounds and the Rafting consolation bracket don't fit in portrait width) — fine in landscape, slightly awkward in portrait.

## How to run

1. Open `index.html` in Safari (Mac or iPad), or serve it via `python3 -m http.server 8000` and open `http://localhost:8000/index.html`.
2. First load shows a "Generate Brackets" button — tap it to seed the three gym tournaments and the Rafting bracket.
3. The Slot view opens to slot 1 (Sunday 1:00). Use the dropdown or arrows to navigate.
4. Tap a team to mark them the winner. Tap the other team to switch; tap "Clear winner" to undo.
5. Tap "Edit" next to "Excluded this slot" to mark teams that can't play that slot.
6. Tap "Swap" under a team to substitute a different team (turns the game into a filler).
7. Open the **Brackets** tab for the per-sport brackets. The **Rafting** tab there is interactive — tap a team to set each match's winner, tap the other to switch, or "Clear" to undo.
8. Open the **Standings** tab for overall ranking, per-team game counts, the points-per-place table, and the seven events. Tap "Edit" on an event to enter placements, times, judge scores, or rankings.
9. In Settings, export the JSON periodically as a backup.
