# Roadmap: 5-seat table

Goal: let the player drop into a full table so every card dealt is visible and has to be counted, like a real casino shoe game.

## Table layout

- Five seats around the dealer. Seat 1 (first base) is dealt first and acts first. Seat 5 (third base) acts last.
- Default: two seats open, three occupied by computer players.
- The player taps any seat to sit. Playing several seats at once is allowed, each with its own bet.
- The player can also take over a computer player's seat or hand, and hand it back.

## Dealing and action order

1. One card to each active seat, left to right (seat 1 to seat 5).
2. Dealer upcard.
3. Second card to each active seat.
4. Dealer hole card, face down.
5. Insurance offer if the dealer shows an ace.
6. Dealer peeks if showing ace or ten.
7. Each seat acts in order. Later seats see earlier hands play out before deciding.
8. Dealer plays, then all seats settle.

Every card that becomes visible, for any seat, updates the running count.

## Computer players ("casino regulars")

- Not perfect strategy. Model a mix of typical habits:
  - Basic-strategy-ish players with occasional mistakes.
  - Conservative players who never hit stiff hands against a weak upcard but also stand on 12 to 16 against 7 or higher too often.
  - "Never bust" players who stand on any 12 or more.
  - Players who mirror the dealer (hit below 17).
  - Players who take insurance on a hunch, rarely split tens, rarely double.
- Varied bet sizes per player, with small random changes between hands.
- Each computer player gets a style assigned when they sit down.

## Computer player turnover (user option)

Setting: **Table turnover**
- **Stay**: the same computer players stay for the whole shoe.
- **Come and go**: between hands, a computer player may leave an occupied seat or a new one may sit in an empty seat. Players should not join or leave mid-hand.

Turnover changes cards used per round, which is what makes the true count drift in practice.

## Settings to add

- Number of computer players at start (0 to 4), default 3.
- Table turnover: Stay or Come and go.
- Starting seat for the player.
- Optional: show or hide computer players' bets and styles.

## Engine changes

- Replace the single `S.hands` list with a `seats` array of 5: `{ occupant: 'open' | 'player' | 'bot', style, bankroll, hands: [] }`.
- Round flow loops over active seats in order, each with its own hands (splits still allowed per seat).
- Player controls act on the currently active player-controlled seat and hand.
- Strategy hints, index plays and strategy accuracy apply only to hands the player controls. Bot hands are not scored.
- Settlement runs per seat. Net result and bankroll track the player's seats only.
- Emergency reshuffle must collect cards from every seat.

## UI changes

- Table view with five seat spots on an arc, dealer at the top.
- Phone layout (about 400px wide) has to stay usable: compact cards, seat spots in an arc, active seat highlighted.
- Bet placement per player seat before the deal. Tap an open seat to sit and bet.
- Visible turn indicator so it is clear whose action it is.

## Acceptance checks

- With 3 bots and 2 open seats, the running count after a round equals the sum of tags over every visible card at the table.
- Hole card is counted only when revealed.
- Playing 3 seats at once works, including splits and doubles on different seats in one round.
- Bots finish their turns without freezing the round, including bust and 21 cases.
- Turnover set to Stay keeps the same bots for the whole shoe. Come and go changes seat occupancy only between hands.
