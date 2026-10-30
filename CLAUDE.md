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
- Hints explain why, not just what: `explain()`, `indexWhy()` and `recIns()` return a `reason`. The guide chart is generated from `basic()` via `chartCode()`, so it always follows the current rules. Keep both in sync when changing strategy.
- `S.seenN` counts every shown card per rank since the shuffle. `checkCount()` asserts the running count equals base plus tags over `seenN`, so the breakdown and the count cannot drift apart. Keep `seenN` updated only through `seen()`.

## Play together (two phones)
- The host phone runs everything (shoe, dealer, computer players, both bankrolls). The guest phone mirrors a `buildView()` snapshot sent after every `render()` and applies it with `applyView()`. The guest never receives the shoe or the hole card, only `pos`, shoe length and `seenN`.
- `seat.owner` is 'h' or 'g'. `human(s)` is any person, `mine(s)` is a seat of the person holding this phone (`S.me`). Game logic that waits for input uses `human`, UI uses `mine`. Computer seats have unlimited funds in `bankOf()`.
- Every button goes through `cmd()` and then `runCmd(m, who)`, which validates the seat owner and phase. The host trusts nothing from the guest beyond that.
- Each phone keeps its own bankroll, stats, counting system and coaching settings. The guest takes the host's rules (`RULE_KEYS`) in memory only, and `save()` writes the guest's own rules back. The guest's running count is recomputed from `seenN` with the guest's own system.
- If the other person drops mid-hand, `dropGuestSeats()` hands their seats to the computer ('book' style, `leaving`) so the hand can finish, and they are cleared at settlement.
- Test hook: `window.__CC_SIG` can replace the ntfy.sh relay.

## Views, stats and drills
- Three tabs share one page: Play (`#vPlay`), Count drill (`#vDrill`) and Stats (`#vStats`). `setView()` switches them and keeps the choice in `cfg.view`.
- `stats` is lifetime and saved. `sess` has the same shape, lives in memory and starts over when the app opens. Change both through `tally(fn)`.
- `recordRound()` adds one entry per person per round to `stats.hist` (capped at `HIST_MAX`) and `sessHist`. The count at the deal comes from `S.start` via `startCount()`, using this phone's own system, so the guest gets it from `roundNet.st`.
- The drills use their own six-deck cards from `shuffle()` and never touch the table shoe, `S.rc` or `S.seenN`. The running count drill starts at 0 for every system.
