# Cut Card Blackjack: notes for Claude Code

Single-file app: `index.html` (CSS, markup, one IIFE script). Keep it dependency-free and dark-felt themed (single visual world, tokens at the top of the `<style>` block). Fonts come from Google Fonts only.

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

## Next task
Implement the 5-seat table described in `docs/ROADMAP.md`.
