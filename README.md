# Polymarket: what does NOT work (4 tests on real data)

**Issue 1, October 2026.** Research by the [OrcaLayer](https://orcalayer.com) team on Polymarket data from September and October 2026.

## Why this exists

Most Polymarket content is some version of "I found a strategy with a 90% win rate". We did the opposite. We took four popular
ways people try to make money on Polymarket, tested each one on real data (a recorded order book for every football market,
on-chain trades, a live detector), and show with numbers **why they do not work**, or only work for whoever is physically fastest.

## Contents

| # | File | Idea | One-line verdict |
|---|---|---|---|
| 1 | [01-cross-market-arbitrage.md](01-cross-market-arbitrage.md) | "Risk-free" arbitrage between linked markets of one match | You can see it, you cannot take it: the exchange holds taker orders for 1 s in football, and in that second the legs get pulled or taken |
| 2 | [02-inplay-maker.md](02-inplay-maker.md) | Market making (resting limit orders) in live football | 13-day replay on the recorded book: **-$23,055**, 12 of 13 days negative |
| 3 | [03-informed-traders-copy.md](03-informed-traders-copy.md) | Copying wallets that buy "right before the goal" | Their edge is real (+$2.33 per $10 out of sample), copying 1-5 s later earns zero; plus a look-ahead trap (+4.23 became -0.41) |
| 4 | [04-calibration-longshot.md](04-calibration-longshot.md) | Polymarket prices are "miscalibrated" (favorite-longshot bias) | 35k markets: an out-of-sample calibration fix improves nothing, the trading rule makes about $0 per $1 |
| 5 | [05-polymarket-mechanics-facts.md](05-polymarket-mechanics-facts.md) | Exchange facts that break most "bots" | Taker delay, fee formula, feed and chain latency, malicious "bot" repos |

## Data (`data/`)

| File | Contents |
|---|---|
| `arb_live_candidates.csv` | 369 candidates from the live arbitrage detector (2026-10-05 16:30 to 2026-10-06 10:07 UTC): event, relation type, `secondsDelay`, edge from WebSocket and from REST at 0 / 1 / 3 s, minimum size. Columns below |
| `maker_replay_daily.csv` | Daily maker replay (Sep 17-29, plus Oct 5 variants): PnL, maker rebate, volume, 30 s / 10 min markout, losses right before goals. `quote_size_shares` = 50 shares per quote, `position_cap_shares` = 200 shares max position per market, `requote_latency_s` = requote delay. `variant_file`: `2026MMDD` is the base variant (on Oct 5 with 0.05 s requote), `_lat03` / `_sticky` / `_improve` are other Oct 5 modes |
| `calibration_by_category.csv` | Calibration by category and horizon: Brier, Platt slope, out-of-sample "raw vs Platt" test, trading rule result |
| `calibration_bins.csv` | Calibration table: price bucket, mean price, share of markets that resolved Yes |

**`arb_live_candidates.csv` columns.**
- `time_utc` - when the WebSocket book showed the candidate. `event_slug`, `market_kind` (the relation), `outcomes` (the legs, in order).
- `seconds_delay` - Gamma `secondsDelay` of the event's markets (1 = taker orders wait 1 s).
- `asks_ws` - best ask of each leg bought, in `outcomes` order, from the WebSocket book. For "exactly one of these happens" groups (1X2;
  "nobody scores" vs. "over 0.5") the detector takes the cheaper of two sets: all Yes legs (pays $1) or all No legs (pays $1 x (legs - 1),
  so $2 for a 1X2). A 1X2 row whose asks sum to about 1.9-2.0 is a No set. For "A implies B" relations (ladders, half <= match totals,
  "0-0 = nobody first" in both directions) it is No on A + Yes on B (pays $1); for "cannot both happen" it is No on both (pays $1).
- `edge_ws` - edge per $1 of payout at that moment: (payout - sum of asks - taker fees) / payout.
- `edge_rest_0s`, `edge_rest_1s`, `edge_rest_3s` - the same edge from a REST book request right away, 1 s and 3 s later.
- `min_size_0s` - shares available on the thinnest leg at the 0 s REST check.
- `confirmed_rest_0s` - True if `edge_rest_0s` > 0.

Data sources: our own Polymarket order book recorder (CLOB WebSocket, every football market since 2026-09-17, 99.9% trade
coverage), on-chain Polygon trades (our own indexer), Polymarket Gamma API and data-api.

## What is not here

- **Wallet addresses.** Where the text says "a wallet", it describes behavior. We do not expose individuals.
- **Code.** Each file describes the method in enough detail to reproduce it. Cleaned-up scripts may come in later issues.

## Caveats

Results come from our setup and our period (September to October 2026). This is not financial advice. Polymarket changes its
rules (fees, delays), so conclusions can go stale.

## License

Text and data: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Use freely, credit "OrcaLayer Research" with a link.
