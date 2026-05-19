# RMCC 2026 Sports Scheduler — Progress

## What we built

A single-file iPad web app that runs three concurrent 12-team double-elimination tournaments (Volleyball, Hockey, Softball) across the 34 thirty-minute sport slots of the RMCC 2026 camp week, plus a camp-wide standings view that combines bracket finishes with optional non-bracket events.

### Files
- `index.html` — the entire app (~2085 lines). Vanilla HTML/CSS/JS, no build step.
- `.gitignore` — excludes the schedule PDF from any future git repo.
- `2026 RMCC Daily Schedule.pdf` — local reference only, never committed.

### Decisions locked in
- **Teams**: generic Team 1–12 (no name customization).
- **Format**: double-elimination, **no bracket reset**, 22 bracket matches + 4 placement matches = 26 real matches per sport.
- **Seeding**: independent random 1–12 per sport.
- **Schedule (34 slots)**: Sun 12 / Mon 9 / Tue 2 / Wed 5 / Thu 0 / Fri 6.
- **Placement**: every team gets a final place 1–12. Losers of LB R1/R2/R3/R4 play placement games for 11/12, 9/10, 7/8, 5/6 instead of just being eliminated.
- **Conflicts**: when a real match can't run (team excluded or playing another sport), defer it and substitute a filler/exhibition with an eliminated team (winner recorded for display only).
- **Grand Finals**: all three GFs must fall in Friday slots **and in three different Friday slots** (6 Friday slots, 3 GFs).
- **Standings points**: editable per-place points table, defaults `[12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1]`. Custom non-bracket events supported with manual place entry; ties give each tied team the higher place's points and skip the next.
- **Persistence**: `localStorage` on a single iPad.

## What's done and verified

| Feature | Status |
|---|---|
| 34-slot schedule definition (Sun–Fri blocks) | ✅ |
| 12-team double-elim bracket generator (22 bracket matches/sport) | ✅ |
| Placement games for clean 1st–12th (4 PL matches/sport) | ✅ |
| Random seeding per sport on bracket generation | ✅ |
| Bracket progression (winner advances; loser drops to LB or PL match) | ✅ |
| Per-team `place` recorded for all 12 teams when bracket completes | ✅ |
| Per-slot scheduler with sport-order rotation for fairness | ✅ |
| No-double-booking rule across the 3 sports per slot | ✅ |
| Per-slot team exclusion (tap to toggle) | ✅ |
| Filler-game substitution when real match can't run | ✅ |
| Filler team selection prefers eliminated teams | ✅ |
| Grand Final gated to Friday slots | ✅ |
| Grand Finals forced into three separate Friday slots | ✅ |
| Tap-to-pick winner with auto-advance | ✅ |
| Clear/correct winner with downstream invalidation (incl. places) | ✅ |
| Manual team swap (downgrades real match to filler) | ✅ |
| Slot view with sport-color icons + sport cards | ✅ |
| Brackets view (per-sport WB/LB/PL/GF visualization with champion banner) | ✅ |
| SVG connector lines drawn between matches within each bracket section | ✅ |
| Sport icons (VB/HK/SB) on slot cards and bracket tabs | ✅ |
| **Standings tab** with overall ranking, per-event breakdown | ✅ |
| Editable points-by-place (1st–12th) | ✅ |
| Custom non-bracket events with manual place assignment | ✅ |
| Per-team game-count grid (per sport + total) | ✅ |
| Settings (re-seed, export JSON, import JSON, full reset) | ✅ |
| localStorage persistence across reloads | ✅ |
| First-run "Generate Brackets" flow | ✅ |
| iPad-tuned CSS (large tap targets, system fonts, viewport meta) | ✅ |

### Notes on persistence
- State version is still `1`. Old saved tournaments retain their pre-placement-games routing (LB losers go straight to ELIM); they will not retroactively gain PL matches. New tournaments (after re-seed or full reset) use the new placement-games flow. `ensureStateFields` backfills `pointsByPlace`, `customEvents`, and per-tournament `places: {}` on load so old saves still render.

## What's left to do

### Required before camp
- **Real iPad smoke test**: open the file in actual iPad Safari (AirDrop or local network), Add to Home Screen, exercise the flow in both portrait and landscape. Browser tests confirm logic but not native gestures, scroll behavior, or keyboard quirks. **Specifically test**: placement games appearing late-week, GF-separation across Friday slots, standings tab editing, custom event creation, bracket connector lines on a small screen.
- **Decide on hosting**: simplest is AirDrop `index.html` to the iPad and open from Files. Alternative is a tiny local web server on a Mac during camp. Pick one and document for whoever runs the iPad.

### Nice-to-haves (not blocking)
- **Recent winners log**: rolling "last 5 results" feed on the slot view for context.
- **Day-level exclusion**: "Team X out all of Tuesday" shortcut. Currently must exclude per-slot.
- **Sport-order rotation visibility**: surface "Volleyball picks first this slot" so the user understands why a particular sport got first dibs on a contested team.
- **Filler game opt-out**: setting to skip filler games entirely (show "—" instead) if the user prefers not to run exhibitions.
- **Print/screenshot view**: a clean printable bracket page for posting at camp.
- **PWA manifest + icons**: makes "Add to Home Screen" produce a proper app icon and splash screen instead of a generic Safari favicon.
- **Cross-section bracket lines**: connector lines currently stay within each bracket section. Cross-section lines (WB loser → LB, WB-F → GF, LB-F → GF, WB-SF → LB-R4, LB losers → PL) are not drawn.

### Known small UX gaps
- No undo for "Full reset" or "Re-seed all brackets" beyond the confirmation modal — JSON export is the only recovery path.
- The brackets view scrolls horizontally on narrow screens (intentional, since 6 LB rounds don't fit in portrait width) — fine on iPad landscape, slightly awkward in portrait.

## How to run

1. Open `index.html` directly in Safari (Mac or iPad). No server needed.
2. First load shows a "Generate Brackets" button — click it to randomly seed all three sports.
3. The Slot view opens to slot 1 (Sunday 1:00). Use the dropdown or arrows to navigate.
4. Tap a team to mark them the winner of that game. Tap the other team to switch; tap "Clear winner" to undo.
5. Tap "Edit" next to "Excluded this slot" to mark teams that can't play this specific slot.
6. Tap "Swap" under a team to substitute a different team (turns the game into a filler; the original bracket match returns to the queue).
7. Open the **Standings** tab anytime to see overall ranking, per-team game counts, edit points-per-place, or add custom events.
8. In Settings, export the JSON periodically as a backup.
