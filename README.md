# Cut Card Blackjack

A single-file blackjack table for practicing card counting. Open `index.html` in a browser. No build step, no dependencies beyond Google Fonts. Play together also talks to the ntfy.sh relay and Google STUN servers, only to connect two phones.

## What it does

- Five-seat table, dealt clockwise from first base (seat 1, on the right of the screen). Unnamed computer players with imperfect habits fill other seats and every card they get is part of the count. Tap any seat between hands to play it, each with its own bet. Optional table turnover lets players come and go between hands.
- Picks up where you left off: bankroll, stats, settings, seats, bets, shoe and running count are saved on this device between hands. Leaving mid-hand restores the table to just before the deal, with your bets back.
- Adjustable shoe: 1 to 8 decks, penetration 50 to 90%, one unseen burn card, cut card reshuffle.
- Shoe shuffled with `crypto.getRandomValues` (Fisher-Yates, rejection sampling, no modulo bias).
- Full rules: hit, stand, double, split (up to 4 hands), late surrender, insurance, dealer peek.
- Configurable rules: dealer S17 or H17, blackjack 3:2 or 6:5, double after split, double on 9 to 11 only, resplit aces.
- Counting systems: Hi-Lo, KO, Hi-Opt I, Zen, Omega II.
- Count hidden by default. Reveal it, or get quizzed on the running count (and optionally the true count) every hand, every few hands, or at random.
- Coaching: tap the lightbulb during your turn for a hint that says what to do and why (dealer bust odds, your bust risk, or the count). Post-decision feedback explains the reason too, and Hi-Lo index plays (Illustrious 18) are included. Settings can show the hint every turn.
- Count breakdown (shown when the count is shown, each part toggles open or closed): every rank seen out of how many are in the shoe, a table adding the tags up to the running count, the share of tens and aces vs small cards still unseen, next-card odds, and a rough estimate of who the shoe favors and by how much. A Shuffle a new shoe button in Settings restarts the count.
- Guide (book icon): rules and actions, a strategy chart for hard, soft and pair hands that follows your current rules, and a counting course covering Hi-Lo, running to true count, bet ramp, index plays and the other systems.
- Play together (people icon in the header): one phone taps Host a table and gets a 5 character code, the other taps Join and types it (or opens the invite link). You share one live table: same shoe, seats, dealer and count, and the host phone deals. Each of you keeps your own bankroll, stats and counting settings on your own phone. Needs an internet connection on both phones, and the page must be served over https (for example GitHub Pages) so the invite link works. If one of you drops, the computer finishes the hand and the code can be used to rejoin.
- Stats persisted in `localStorage`: hands, net, count accuracy and streak, strategy accuracy.

## Run it

```
open index.html
```

Or serve it locally: `python3 -m http.server 8000` and visit http://localhost:8000.

## Structure

Everything lives in `index.html`: styles, markup, and one script. The script is organized as engine (shoe, counting, strategy), rendering, then round flow and events.

## Next up

[docs/ROADMAP.md](docs/ROADMAP.md) describes the 5-seat table, which is implemented. Two deliberate changes from it: computer players are unnamed and unlabeled, and any seat you tap is simply yours (no taking over a computer player's hand mid-round).

This is a practice tool, not gambling advice.
