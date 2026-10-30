# Cut Card Blackjack: notes for Claude Code

Single-file app: `index.html` (CSS, markup, one IIFE script). Keep it dependency-free and dark-felt themed (single visual world, tokens at the top of the `<style>` block). Fonts come from Google Fonts only. No script libraries. Play together uses two network services but no code from them: ntfy.sh (a public relay, used only to swap WebRTC connection details) and Google STUN servers.

## Conventions
- Avoid em dashes in UI copy and docs.
- Mobile first (about 400px wide). No horizontal page scroll. Keep side gutters at 16px minimum.
- Use design tokens in `:root` instead of literal colors.
- Settings and stats persist in `localStorage` under the key `cutcard.v1`. Wrap storage access in try/catch.
- Randomness must come from `crypto.getRandomValues` via `rnd()` and `shuffle()`.

## Counting rules that must stay correct
- Count only cards the player can see. The dealer hole card is counted when it is revealed.
- The burn card is never counted.
- Unbalanced systems (KO) start at `4 - 4 * decks`.
- True count uses decks remaining rounded to the nearest half deck.

## State
- Saved between hands only (see `save()`, `snapshot()`, `restoreTable()`). Leaving mid-hand restores the table to just before the deal.
- Seat 1 is first base and is dealt first. It is drawn on the right of the screen, so play runs clockwise.
- Computer players are unnamed. Any seat the player taps is theirs.
- Hints explain why, not just what: `explain()`, `indexWhy()` and `recIns()` return a `reason`. The guide chart is generated from `basic()` via `chartCode()`, so it always follows the current rules. Keep both in sync when changing strategy. Every hint card also shows the count it used (`countNote()`, same math as `recommend()` and `recIns()`), even when the count is hidden.
- A new shoe starts uncut (`S.uncut`). At the next deal, `deal()` announces it and either asks for the player's cut (`askCut()`, the `#cutOv` sheet) or cuts at random when `cfg.cutBy` is 'auto'. `applyCut()` moves the cards above the cut to the back before any card is seen, so the count is untouched. In Play together the host cuts.
- `S.seenN` counts every shown card per rank since the shuffle. `checkCount()` asserts the running count equals base plus tags over `seenN`, so the breakdown and the count cannot drift apart. Keep `seenN` updated only through `seen()`.

## Play together (two phones)
- The host phone runs everything (shoe, dealer, computer players, both bankrolls). The guest phone mirrors a `buildView()` snapshot sent after every `render()` and applies it with `applyView()`. The guest never receives the shoe or the hole card, only `pos`, shoe length and `seenN`.
- `seat.owner` is 'h' or 'g'. `human(s)` is any person, `mine(s)` is a seat of the person holding this phone (`S.me`). Game logic that waits for input uses `human`, UI uses `mine`. Computer seats have unlimited funds in `bankOf()`.
- Every button goes through `cmd()` and then `runCmd(m, who)`, which validates the seat owner and phase. The host trusts nothing from the guest beyond that.
- Each phone keeps its own bankroll, stats, counting system and coaching settings. The guest takes the host's rules (`RULE_KEYS`) in memory only, and `save()` writes the guest's own rules back. The guest's running count is recomputed from `seenN` with the guest's own system.
- If the other person drops mid-hand, `dropGuestSeats()` hands their seats to the computer ('book' style, `leaving`) so the hand can finish, and they are cleared at settlement.
- Test hook: `window.__CC_SIG` can replace the ntfy.sh relay.

## Views, stats and drills
- Four tabs share one page: Solo and Together (both `#vPlay`), Drill (`#vDrill`) and Stats (`#vStats`). `setView()` switches them and keeps the choice in `cfg.view`.
- The tab bar is fixed at the bottom of the screen, under the thumbs. The body keeps bottom padding for it, and `#tabNote` messages sit just above it.
- During a hand, the controls panel shows the hand being decided next to the dealer's up card (`handVsHTML()`), so a decision never needs the table in view. `revealControls()` scrolls the buttons into view once when it becomes your move.
- Long screens are split into folding sections: `grpOpen()` in Settings and `fold()` in Drill and Stats, each with a one-line summary when closed. `openGrps` remembers which are open for the visit.
- `#intro` is a one-time welcome card on the Solo table, dismissed into `cfg.introDone`.
- Solo and Together are separate tables. `S.mode` is 'solo' or 'tog'. Each has its own saved table, bankroll and stats in `slots` and `ST`. `switchMode()` puts one table away and restores the other, only between hands and never while hosting or joined. Play together lives in the Together tab (`#togPanel`), and the guest sends their Together bankroll.
- Per table, `ST[mode].life` is lifetime and saved (`stats` for solo, `tstats` for Together). `ST[mode].sess` has the same shape, lives in memory and starts over when the app opens. Change both through `tally(fn)`, which follows `S.mode`.
- `recordRound()` adds one entry per person per round to the current table's `hist` (capped at `HIST_MAX`). The count at the deal comes from `S.start` via `startCount()`, using this phone's own system, so the guest gets it from `roundNet.st`.
- The drills use their own cards from `shuffle()` and never touch the table shoe, `S.rc` or `S.seenN`. The running count drill starts at 0 for every system. Whole-deck running count runs hold 1 to 3 cards back face down (`DR.held`) so the answer is not always the deck total. Drill totals and the drill log are in `dstats` and `dsess`, changed through `dtally(fn)` and `logDrill()`.
- Each run copies its settings into `DR` (`kind`, `per`, `pace`, `decks`, `lvl`), so drill code reads `DR`, not `cfg`. Pace is reported as seconds per deck (`deckSecs()`). Card values answer on pointerdown and show the next card at once, with right or wrong for the previous card in `DR.last`, so there is no feedback pause on the clock. The two-thumb pad (`tagPadHTML()`, `cfg.tagPad`) puts minus values on the left, plus on the right and a 0 under each thumb.
- Campaign: `LEVELS` are fixed drills with a benchmark. Passing the current level `PASS_NEED` times in a row unlocks the next. Progress is `dstats.camp` (`lvl`, `prog`), kept when drill stats are reset. Replaying a cleared level is practice and does not change progress.
- Saved setups (`presets`) are named copies of the `PKEYS` settings. At someone else's table, applying one puts rule settings in `Net.ownRules` only.
- The bet size suggestion is the `betHint` setting, not a lightbulb.
- Counts are typed on an in-app number pad (`numPadHTML()`, `padKey()`), never a text input, so the phone keyboard never opens. Used by the table count quiz (running and true count, tap a box to pick which) and the running count drill answer. Hardware keys work too: digits, minus, Backspace, Enter.
