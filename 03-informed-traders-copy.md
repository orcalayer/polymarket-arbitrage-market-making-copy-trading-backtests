# 3. Pre-goal "insiders" are real. Copying them in-play is useless

## The idea

On Polymarket football markets you can see wallets that regularly buy the side that is about to score **before** the price jumps.
Find them, copy them. Sounds like a strategy.

## Part A. The look-ahead trap: +4.23 became -0.41

Our goal detector (a book jump across several linked markets of a match at once) fired 481 times (late September to early October).
In 140 of those, one of the "goal leaders" had bought >= 3 s BEFORE the book moved (median 22 s before).

- The detector itself, entering 1 s after its signal on those 140: **-$2.08 per $10** (t -3.0). The market had already repriced.
- Copying the leader 1-5 s after their buy: **+$4.23 per $10 (t 2.1, n 95)**. Looks like a find.

**It is not.** We selected exactly the buys that were followed by a goal. In real time you do not know that.

The honest test: take **every** football taker buy by these 94 leaders over a week (price 0.10-0.90), without knowing whether a goal
follows (13,794 signals),
and copy 1-5 s later at real trade prices + 1 c:

| | Copy 1-5 s later | At their own price |
|---|---|---|
| all | **-$0.41 per $10** (t -2.8, n 6,243, -$2,584) | +0.08 (n 13,794) |
| in-play | -0.42 (t -2.5, n 4,990) | +0.22 (t 1.7) |
| pre-match | -0.40 (n 1,253) | -0.28 |

The find is gone. Lesson: **any selection "by outcome" must be re-tested on all signals, with no knowledge of the outcome.**

## Part B. A real edge that cannot be copied

Second, honest attempt. 1,224 "goal moments" (at most one per match per 60 s), of which **793 were real goals**: 553 in-sample
(Sep 17-29), 240 in the test week (Sep 30 to Oct 6, international matches).

Selection on the in-sample period: the wallet was positioned on the scoring side within 60 s before the book moved at least 3 times,
and three times more often "for" than "against". That gives **92 wallets**. Test: the following week, on all of their football buys:

| | Result per $10 | t | n |
|---|---|---|---|
| **at their own price** | **+2.33** | **6.8** | 10,198 (+$23,752) |
| copy 1-5 s later | **-0.10** | -0.5 | 6,210 |

The edge is **real and out of sample** (t 6.8 is not noise). But a copy 1-5 s later earns zero.

Why, shown second by second on one of the best such wallets (1,058 buys, book recorded to the millisecond):
- 92% of its buys are **maker** fills (the exchange does not delay makers, unlike takers, see file 5);
- in a quarter of cases its fill is the last trade at the old price: the ask is +9 c **in the same millisecond**, +14 c a second later;
- copying 2.5 s later in those cases: -$0.67 per $10; overall the copy makes +0.16 (t 0.4) against the wallet's own +2.77 (t 3.9).

## Conclusion

Polymarket has participants who consistently know or see a goal before the market does, and the data shows it with high
confidence. But their entire edge sits in the seconds **before** the price moves, and they are often makers (no exchange delay).
Anyone copying after their trade buys at the new price. Copying "insiders" in live sports is dead.

## What this does not say

This test covers **in-play football on a horizon of seconds**. It does not say that following informed money is useless in general.
Where information takes hours or days to get priced in (politics, long-dated markets), the copy delay is a much
smaller share of the move. We have not tested that here, and it is the subject of a future issue.

## Caveats

Two weeks of football (September to October 2026, including an international week). Copy price = mean price of real trades in the
+1-5 s window plus 1 c, taker fee included, held to settlement. Wallet addresses are deliberately not published.
