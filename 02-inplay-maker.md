# 2. Market making in live football: -$23k in 13 days

## The idea

Takers pay fees, makers do not, and in sports makers even get part of the fees back (a 15% rebate). So the logical move is to
quote both sides of the book during the match and earn the spread.

## What we did

**A replay on the recorded order book.** Our recorder captures the book of every Polymarket football market (WebSocket,
`price_change` / `book` / `last_trade_price` events, 10-minute files). Before the replay we checked the recording itself:
**99.9% of on-chain trades are present** (3,338 of 3,340 in a 10-minute control window), and 97.6% of trades printed at the
recorded best price.

The maker model (conservative, not tuned in our favor):
- 1X2 markets, in-play only; we quote the best price on both sides, **50 shares per quote**, when the spread is <= 3 c and the
  price is between 0.08 and 0.92;
- we sit **at the back of the queue** at our level; the queue only moves on trades at our price; a trade through our price = full fill;
- requote delay 0.3 s; **position capped at 200 shares per market** (beyond that we only quote the reducing side); held to
  settlement; +15% maker rebate (the sports rate, [fees docs](https://docs.polymarket.com/trading/fees.md));
- two modes: **kill** pulls quotes on a goal-detector signal (+0.2 s, 60 s pause), **nokill** does not.

## Numbers (`data/maker_replay_daily.csv`)

**13 days, 2026-09-17 to 09-29:**

| Mode | Total PnL | Losing days | Shares filled | 30 s markout |
|---|---|---|---|---|
| kill (pull on goal) | **-$23,055** | 12 of 13 | 1.89M | -0.8 to -2.5 c every day |
| nokill | **-$24,110** | 13 of 13 | 2.12M | |

Variants on 2026-10-05 (43 matches, kill / nokill): faster requote (0.05 s) -$1,888 / -$1,594; "sticky" quotes -$1,907 / -$1,761;
"1 c inside the spread, first in queue" -$1,949 / -$2,095. **No variant is positive.**

## Why

We broke down every maker fill on the market (162 matches, 2026-09-30 to 10-06, maker side):

| Moment | Shares | 30 s markout |
|---|---|---|
| normal play | 28.9M | **+0.04 c** |
| 20 s before a goal signal | 0.55M | **-4.33 c** (-$22.9k) |
| 60 s after the signal | 2.83M | **-1.37 c** (-$37.0k) |

- In normal play makers earn pennies (+0.04 c per share), and those are the makers **at the front of the queue**.
- An order at the back of the queue only fills when price moves through it, i.e. right before a level breaks (a goal, a dangerous
  attack). Classic adverse selection: you get filled exactly when you are already wrong.
- The goal kill-switch does not help: the detector sees the goal from the book moving, and most replay losses happen in normal play.
- Liquidity rewards on football are tiny: 50 share minimum, spread <= 4.5 c, $1-10 per day per market (Gamma `rewardsDailyRate`).

## Conclusion

In-play football market making is a business for whoever sits first in the queue and pulls quotes in milliseconds. For everyone
else it is a steady loss of about 1-2 c on every share filled.

## Caveats

The queue model is approximate (we cannot see exact queue position, so we put ourselves at the back). The replay covers 1X2 only
and in-play only. Pre-match market making and non-football markets may behave differently.
