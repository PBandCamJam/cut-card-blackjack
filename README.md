# Cut Card Blackjack

A single-file blackjack table for practicing card counting. Open `index.html` in a browser. No build step, no dependencies beyond Google Fonts.

## What it does

- Adjustable shoe: 1 to 8 decks, penetration 50 to 90%, one unseen burn card, cut card reshuffle.
- Shoe shuffled with `crypto.getRandomValues` (Fisher-Yates, rejection sampling, no modulo bias).
- Full rules: hit, stand, double, split (up to 4 hands), late surrender, insurance, dealer peek.
- Configurable rules: dealer S17 or H17, blackjack 3:2 or 6:5, double after split, double on 9 to 11 only, resplit aces.
- Counting systems: Hi-Lo, KO, Hi-Opt I, Zen, Omega II.
- Count hidden by default. Reveal it, or get quizzed on the running count (and optionally the true count) every hand, every few hands, or at random.
- Coaching: basic strategy hints, post-decision feedback, Hi-Lo index plays (Illustrious 18).
- Stats persisted in `localStorage`: hands, net, count accuracy and streak, strategy accuracy.

## Run it

```
open index.html
```

Or serve it locally: `python3 -m http.server 8000` and visit http://localhost:8000.

## Structure

Everything lives in `index.html`: styles, markup, and one script. The script is organized as engine (shoe, counting, strategy), rendering, then round flow and events.

## Next up

See [docs/ROADMAP.md](docs/ROADMAP.md) for the 5-seat table feature.

This is a practice tool, not gambling advice.
