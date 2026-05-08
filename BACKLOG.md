# Poker Tools — Backlog

---

## Dashboard

### Completed
- [x] Google Sheets CSV data integration with CORS proxy fallback chain
- [x] Password-gated access (sessionStorage)
- [x] KPI stat cards (profit, hourly rate, sessions, win rate, avg hours, best/worst)
- [x] Cumulative profit chart with hourly rate toggle (Chart.js)
- [x] Multiselect filter dropdowns (location, stakes, game type) with select all/none
- [x] Filters show only options available given other active filters
- [x] Date range filter
- [x] Paginated session table (25 rows)
- [x] Responsive dark theme layout
- [x] Fallback CORS proxies for reliable data fetching

### Features (Future)
_(none planned)_

### Bugs
_(none known)_

### Tech Debt
- Password hardcoded in source — fine for now, but not real auth
- CORS proxy dependency — if all three proxies go down, dashboard breaks

---

## Driller

### Completed
- [x] Core deck engine with shuffle and burn cards
- [x] Range-based hero dealing (preflop raising ranges from PreflopTrainer, per spot)
- [x] Game state machine (street progression, pot/stack tracking)
- [x] 7 preflop spot definitions with correct pot math
- [x] Config screen with mutual exclusion (4BP/100bb, Limp/IP lock)
- [x] Position-aware opponent logic (IP: V checks/calls; OOP: V checks/bets 1/3 or 2/3 pot when checked to, calls when bet into)
- [x] User action buttons (check/bet/call/fold/raise + freeform sizing)
- [x] Card rendering with suit colors
- [x] Hand summary screen with full action log
- [x] Session log drawer (in-memory)
- [x] Responsive layout (mobile + desktop)
- [x] Landing page with navigation to dashboard + driller
- [x] Dark theme matching dashboard design system
- [x] Stakes selector ($1/$3, $2/$5, $5/$10) with dollar-based UI
- [x] 8-max poker table UI with oval felt and positioned seats
- [x] Simplified seat displays with active-turn glow indicators
- [x] Hero hole cards displayed below table
- [x] Animations: preflop sequencer, board card dealing, opponent action pulse
- [x] Preflop animation sequencer: step-by-step replay of blinds, folds, bets before flop is dealt
- [x] Chip pill system: bet/raise/call amounts shown as pills at seats, collected into center pot pill on street end
- [x] Facedown card indicators at active seats, slide-away on fold
- [x] Active-turn glow pulse animation for action indicator
- [x] Visual overhaul to match PreflopTrainer: pure black theme, green felt with brown rail, status-colored seat borders, responsive stadium table (vertical mobile/horizontal desktop), orange pot pills, white bet pills, PreflopTrainer-style action buttons (red/green/gray), faster animation timings (~300ms actions), white glow active seat indicator, navy card backs, smaller cards matching iOS app sizes
- [x] Mobile UX improvements: larger player circles (62px), balanced card rank/suit sizing, Safari dvh viewport fix with safe-area padding, numeric keypad for bet input with zoom prevention
- [x] Mobile card/fold polish: scaled hero + community cards to match seat height, facedown cards tucked behind seat circles, folded cards disappear completely (no dotted outlines)
- [x] Combined setup + drill screen: config sidebar on desktop, collapsible strip on mobile, persistent "New Hand" button, no auto-summary (history via drawer only)
- [x] Preflop/postflop action flow polish: each action now displays in-seat as a single sequential step (no separate pre-action highlight jump)
- [x] Table/seat UX polish: side-seat hole cards now anchor inward toward table center, and stadium seat placement uses inset geometry to avoid off-screen circles/cards
- [x] Mobile table layout tune: switched to tighter vertical stadium ratio (0.58:1) and slightly smaller play-mode seats (58px) for better fit with controls
- [x] Desktop rail alignment polish: seat inset tuned so player circles sit on the felt rail rather than floating inward
- [x] Layering/spacing polish: hero bet pills render above cards, and opponent card anchors shifted so cards no longer cover player circles
- [x] Reference emulation pass: desktop rail now crosses near seat centers and cards anchor to each seat's outer side (away from table interior)
- [x] Opponent card-back polish: desktop facedown cards are tucked behind seat/ring and overlap slightly to match reference compact stacking
- [x] Layering correction pass: hero + opponent cards now render below seat and active ring overlays; opponent hidden card overlap increased to tighter stack
- [x] Layer order hardening: desktop seat layers now follow explicit stack (table < cards < seat < text < active ring < HERO label)
- [x] Mobile reference-emulation pass: rail-centered seats, centered side-seat text, and compact side hidden-card markers replacing bunny-ear styling
- [x] Pot math + bet sizing audit: corrected 4BP_OOP pot (101 → 101.5bb), fixed SB seat-bet pill overcount on 'raises to' actions, aligned session-log pot rounding with in-game pot pill, villain postflop bets now round to $5 increments regardless of stakes (hero sizing remains free-form)
- [x] Shot clock — optional per-decision countdown timer (Off/On + 5–300s seconds, default 30s/off, persisted to localStorage). Pill displays at top of felt above community cards, resets on each hero decision, stops at 0 and turns red. Reference only — never blocks input or auto-folds.

### Features (Future)

#### NEXT UP — Hand history navigation (back / forward + branch-and-replay) — **fully spec'd, ready to implement**
Companion feature to the shot clock. Lets the user step backward through the current hand to inspect prior decisions ("what was the flop bet again?") and optionally branch and replay from any point. Specs already approved by product owner — no clarifying questions needed.

**User-facing behavior**
- Two new buttons in the control bar next to "New Hand": `◀ Back` and `Forward ▶`. Same on desktop (sidebar) and mobile (play-mode control bar).
- Back/forward step through *every* action and card deal in the current hand — one snapshot per: flop dealt, each villain action shown (pill visible), each hero action shown (pill visible), turn dealt, river dealt, hand end. (NOT during preflop animation — earliest rewind point is "after flop dealt".)
- Restoration is **silent / no animation** — DOM just re-renders to the snapshot's visual state. Preflop animation is never replayed.
- Action buttons remain active at any snapshot.
- If hero takes an action while not at the latest snapshot → **branch**:
  1. Truncate `game.history` to current index.
  2. Reshuffle the remaining deck (so future cards are a fresh runout — per product spec).
  3. Execute the action through normal `handleUserAction` flow with full animations. Villain RNG re-rolls naturally.
- After hand ends, back/forward still work for review. Branching from the end re-engages the hand. New Hand resets history.

**Snapshot model** (all in `driller/index.html`)
- Add `game.history = []` (array of snapshots) and `game.historyIndex` (pointer to live state) inside `startHand()`.
- Snapshot deep-copies (use `structuredClone`): `deck, board, pot, displayPot, userStack, oppStack, streetIndex, actions, streetActions, seatStates, seatBets, holeCards, heroCheckedBackLastStreet, allIn, userFolded, pendingAction, renderedBoardCount, preflopAnimating`.
- Helper: `function snapshot() { game.history.push(structuredClone(state-fields)); game.historyIndex = game.history.length - 1; updateHistoryButtons(); }`.
- Helper: `function restoreSnapshot(idx) { Object.assign(game, structuredClone(game.history[idx])); game.historyIndex = idx; renderHand(); renderPotPill(); renderTableSeats(); renderActionButtons(game.pendingAction); updateHistoryButtons(); stopShotClock(); }`.

**Where to drop snapshot calls**
- In `playPreflopSequence()` — after the flop is dealt (very last step before yielding control to `startStreetAction`).
- In `doOpponentAction()` — at the end of each branch where villain has acted (after `showSeatAction` and any state mutations are complete; before `await advanceStreet()` or `promptUserAction`).
- In `handleUserAction()` — at the end of each branch (after `showSeatAction`, before `advanceStreet` / `doOpponentAction` / `endHand`).
- In `advanceStreet()` — after each new board card is dealt and rendered, before `startStreetAction` runs.
- In `endHand()` — at the end (so the final state is its own snapshot).

**Branch trigger — modify `handleUserAction`**
At the very top, after `stopShotClock()`:
```js
if (game.history && game.historyIndex < game.history.length - 1) {
    game.history = game.history.slice(0, game.historyIndex + 1);
    game.deck = shuffle(game.deck); // fresh runout for new cards
}
```

**Controls / UI**
- Add two `<button>` elements in `#controlBar` (mobile) and the sidebar `.sidebar` block (desktop) next to the existing New Hand button — wire IDs `btnHistoryBack` / `btnHistoryForward`. Reuse the `.new-hand-btn` styling family but smaller (icon-only `◀` / `▶` is fine).
- `updateHistoryButtons()` function: disabled if `!game || !game.history || game.history.length === 0`; back disabled if `historyIndex === 0`; forward disabled if `historyIndex === game.history.length - 1`.
- Listeners: back → `restoreSnapshot(historyIndex - 1)`; forward → `restoreSnapshot(historyIndex + 1)`.

**Edge cases to handle**
- **Animations in flight**: disable both nav buttons while `game.preflopAnimating === true` or during `await delay(...)` calls in `advanceStreet`. Simplest implementation: a module-level `let navLocked = false` toggled around await chains; `updateHistoryButtons` reads it.
- **Preflop**: snapshots only start once flop is dealt. Back at index 0 = "right after flop deal, before any flop action." Cannot rewind into preflop animation.
- **Showdown / fold**: `endHand` snapshot is the final entry. Back from there returns to the last hero decision; user can branch a different action.
- **Shot clock**: `restoreSnapshot` calls `stopShotClock()`. When the user takes an action post-rewind (branch), the normal `handleUserAction` → next `promptUserAction` will restart it.
- **`rerollStreet` interaction**: leave it alone for now — it predates history. If user rerolls, just clear `game.history` and start fresh from the new flop. (Add a snapshot at the end of rerollStreet to seed the new history.)

**Testing checklist (manual, browser)**
1. Start hand. Play through to river. Click Back several times — board cards, pot pill, bet pills, action buttons should all match each prior state.
2. Forward should walk back to the live state.
3. Rewind to flop, take a different action. Confirm: history truncated, villain re-rolls, turn/river are different cards on subsequent runs.
4. Rewind to mid-flop, click Forward repeatedly — should walk through villain action → hero action → turn deal exactly as they happened.
5. End hand by folding flop → Back to flop decision → take call instead → hand should continue.
6. Disabled states: at index 0 Back is disabled; at latest Forward is disabled.
7. Mobile: confirm buttons fit in control bar; tap targets adequate.
8. New Hand clears history (Back disabled).

**Doc updates required in same commit**
- `PROJECT.md`: add a "History Navigation" bullet under Driller Core Modules describing snapshot model + branch behavior.
- `BACKLOG.md`: move this entry to Completed.

---

- [ ] Street navigation controls (go back a street, re-deal turn/river cards) — *partially superseded by hand history navigation above*
- [ ] Configurable preflop sizings
- [ ] Multiway pots (3+ players)
- [ ] Persistent session history (localStorage)
- [ ] Hand export (copy to clipboard as text)
- [ ] Voice/text notes per street for thought process recording
- [ ] Quick-sizing buttons (1/3 pot, 1/2 pot, 2/3 pot, pot, overbet)
- [ ] Villain player types — assign profiles (e.g. tight/passive, loose/aggressive, calling station) that change opponent betting/calling behavior

### Bugs
_(none known)_

### Tech Debt
- Single-file architecture will need splitting if app grows significantly
