# 1. "Risk-free" arbitrage between linked markets of one match

## The idea

Polymarket opens dozens of logically linked markets for a single football match. Relations our detector watched:

- **1X2:** "Team A wins" + "Draw" + "Team B wins". Exactly one resolves Yes, so the three Yes prices must sum to 1.
- "0-0" is the same event as "no team scores first"; "nobody scores" is the same as "total under 0.5".
- half total <= match total; team total <= match total; "both teams score in the half" implies "both teams score in the match"; total "ladders".

If, say, the three 1X2 asks add up to less than 1, you buy all three legs and are guaranteed 1 at settlement. No risk.

## What we did

A live detector ran from 2026-10-05 16:30 to 2026-10-06 10:07 UTC (about 18 h). It listened to the order book WebSocket for
every market of each match, looked for violated relations, re-checked every candidate against the REST order book (at 0, 1 and
3 s) and counted how many shares could actually be taken.

## Numbers (`data/arb_live_candidates.csv`)

- **369 candidates across 35 events**; 262 confirmed by an immediate REST request (so not a feed glitch).
- Edge among the 262 confirmed: median **0.39 c** per set; 30 candidates at >= 2 c. Size on the thinnest leg: median **10 shares**.
  The six largest confirmed edges (12-57 c) all sat on tiny size: 0.09-40 shares on the thinnest leg.
- Most striking case (1X2, Argentina, CSV row 2026-10-06 01:00:46 UTC): the WebSocket book showed a small edge (asks 0.50 / 0.28 /
  0.16), but the REST re-check found one team's Yes offered at 0.34 (fair about 0.55: 0.50 on the WebSocket, 0.57 one second
  later), 1,172 shares on that leg. The three legs summed to 0.77, about 20 c of edge after fees (`edge_rest_0s` 0.2023). At the
  first REST probe the draw leg had only 10 shares (`min_size_0s` 10); 0.2 s later it refilled to ~840, and up to ~400 full sets
  were available for about half a second. By 1 s the 0.34 offer was gone. The CSV keeps only the 0 s probe; the 0.2 / 0.5 s probes
  are in the detector's raw log.
- In theory (taking everything that was in the book): about $50 over ~18 h at <= 20 shares per leg, about $130 at <= 200.
- After 1 s the edge was still positive in 167 of 259 confirmed candidates; after 3 s in 124 of 254. So a "window" seems to exist.

## Why it does not work

**The exchange delays takers.** Gamma API gives every sports market a `secondsDelay` field (a market field, not an event
field: in the event JSON it sits on each of the event's markets)
([Gamma API schema](https://docs.polymarket.com/api-spec/gamma-openapi.yaml)): football **1**, NBA **0**. You can check it
directly: [a football match](https://gamma-api.polymarket.com/events?slug=fif-sri-mri-2026-10-05) (`"secondsDelay":1`) and
[an NBA game](https://gamma-api.polymarket.com/events?slug=nba-lal-sac-2026-10-05) (`"secondsDelay":0`).

Polymarket docs ([Order Lifecycle](https://docs.polymarket.com/concepts/order-lifecycle)) describe a taker delay before matching
on some markets: "Sports/game delay: enabled on configured sports markets around live game conditions", and 250 ms on certain
crypto and finance up/down markets. **"During either delay, the order is pending and cannot be canceled."** Resting limit orders
can still be cancelled as usual, and that is exactly how legs disappear (below: makers themselves pulled 17 of the 41 legs that vanished).

- Of the 35 events, 30 had a 1 s delay and 5 had 0 (NBA).
- **All 30 candidates with >= 2 c edge were in markets with a 1 s delay.** Only 8 of them still had positive edge after 1 s.
- What would still be in the book after 1 s, for the >= 2 c candidates: all legs present in 3 (worth $1.83 combined); some legs
  present in 19 (you would end up holding naked legs, which is no longer arbitrage but a $137 bet); no legs in 6.
- **Sequential execution** (send the riskiest leg first, the rest only if it fills): even an oracle that knows in advance which
  leg will vanish completes 2 full sets out of 28 ($0.72). In 25 the first leg simply does not fill.
- **Before kick-off** (no delay): 82 candidates, all would have filled, but the edge is tiny: **$1.55 combined** at up to $10 per leg,
  none at >= 2 c. (Separate pass: candidate time vs. Gamma `gameStartTime`; the CSV has no pre-match / in-play column.)
- **Who takes the vanished legs** (on-chain trades at the ask +/- 0.5 c within 10 s): of 41 vanished legs, other takers took 24,
  makers pulled 17. A "pure race" (fastest wins) applies to 11 of 27 candidates, with a ceiling of $13.8 over 15 h if you win EVERY one.

## Conclusion

Cross-market arbitrage on Polymarket is **visible every day**, which is why "scanners" show it. But in sports the exchange
deliberately holds taker orders for 1 s, and in that second makers pull bad prices or faster takers take them. Where there is no delay (pre-match), the
edge is pennies. The rest goes to whoever wins the race for those legs, and even someone who won every race would have made a few dollars.

## Caveats

18 hours of live detection, mostly football (Nations League, Argentina, CONCACAF) and NBA; 1X2 accounts for 162 of 369 candidates.
Other sports and longer periods may give different numbers, but the mechanism (taker delay) is the same for every market with
`secondsDelay` > 0.
