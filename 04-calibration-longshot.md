# 4. "Polymarket prices are miscalibrated": a test on 35,000 markets

## The idea

A betting classic: longshots are systematically overpriced and favorites underpriced (favorite-longshot bias). If Polymarket prices
are badly calibrated, you can fit a correction (for example Platt scaling) and trade the difference. The idea is pushed by
"longshot"-style tools and by LLM forecasters in the style of AIA.

## What we did

- Sample: 40,000 closed Polymarket markets, **35,135** with trades; categories from Gamma (sports by league, crypto, other).
- Market price 24 h, 6 h, 1 h and 10 min before the last trade, against whether the market resolved Yes.
- **Time split:** train on markets whose last trade was before 2026-08-11 (60%), test on the rest. No shuffling.
- Metrics: Brier score; Platt slope (1 = perfect calibration); test Brier "raw prices vs after Platt correction"; a trading rule
  (buy where the correction says "underpriced"), test result in $ per $1 after fees.

## Numbers (`data/calibration_by_category.csv`, `data/calibration_bins.csv`)

| Horizon | Markets | Test Brier raw -> Platt | Rule on test |
|---|---|---|---|
| 24 h | 3,506 | 0.1543 -> 0.1547 | -$0.037 per $1 (t -0.7) |
| 6 h | 6,443 | 0.1685 -> 0.1685 | no trades |
| 1 h | 7,364 | 0.1482 -> 0.1479 | +0.008 (t +0.8) |
| 10 min | 8,701 | 0.1749 -> 0.1748 | -0.001 (t -0.1) |

- The calibration correction **improves nothing out of sample** (the Brier difference is in the fourth decimal).
- A rule trained on the past makes **about zero** on the test set (no all-market t-stat even reaches 1; the strongest single-category
  result is negative: 1 h, "uncategorized", -$0.094 per $1, t -2.2).
- Some categories show "nice" slopes (baseball 0.60 at 6 h, for example), but on the test set this does not turn into money.
- Bias only shows up right at the close (10 min before the end, markets priced 0.05-0.10 resolve Yes only 1.9% of the time at a mean
  price of 7.3%). Those are tiny amounts at prices you cannot fill profitably after fees.

## Conclusion

On liquid Polymarket markets prices are calibrated **well enough** that a systematic correction earns nothing after fees. This is consistent with
the AIA Forecaster report by Bridgewater AIA Labs ([arXiv 2511.07678](https://arxiv.org/abs/2511.07678)): on liquid markets the LLM
forecast is worse than the market (Brier 0.1258 vs 0.1106), and the optimal blend (weights fitted by regression: about 2/3 market,
1/3 LLM) reaches Brier 0.106, by our arithmetic only about 4% better than the market, not enough to cover spread and fees. This is
on their liquid-market set (MarketLiquid).

## Caveats

Price = last trade at the horizon (not book mid). Only markets with trades; very thin markets may behave differently.
