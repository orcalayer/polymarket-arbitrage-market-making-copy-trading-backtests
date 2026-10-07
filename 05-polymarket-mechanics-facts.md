# 5. Polymarket mechanics that break most "bots"

Measured or checked by us in September and October 2026. Polymarket changes its rules, check the current docs.

## Taker delay in sports (`secondsDelay`)

- Gamma API gives every sports market a `secondsDelay` field (a market field, not an event field) ([Gamma API schema](https://docs.polymarket.com/api-spec/gamma-openapi.yaml)):
  **football 1 s, NBA 0** (examples: [football](https://gamma-api.polymarket.com/events?slug=fif-sri-mri-2026-10-05),
  [NBA](https://gamma-api.polymarket.com/events?slug=nba-lal-sac-2026-10-05)).
- Docs ([Order Lifecycle](https://docs.polymarket.com/concepts/order-lifecycle)): "Sports/game delay: enabled on configured sports
  markets around live game conditions"; certain crypto and finance up/down markets have a 250 ms taker delay;
  **"During either delay, the order is pending and cannot be canceled."** Limit orders already resting in the book can be cancelled as usual.
- Consequence: any "see a bad price, hit it as taker" strategy in football loses to whoever removes that price within the second.
  See files 1 and 3.

## Fees

- Only takers pay ([fees docs](https://docs.polymarket.com/trading/fees.md)): **fee = C x feeRate x p x (1 - p)**, where C is the
  number of shares and p the price. Rates (also in Gamma `feeSchedule.rate`): crypto 0.07, sports 0.05, economics 0.05,
  politics 0.04, finance 0.04, geopolitics 0 (full table at the link).
- Most expensive near 0.50 (p(1 - p) = 0.25), cheapest near 0 and 1.
- Makers pay no fee and receive part of taker fees (maker rebate): sports 15%, crypto 20%, most others 25%.

## Data latency

- The order book WebSocket lags the exchange by **17 ms** on average. The feed is fast.
- On-chain settlement (Polygon) arrives **2.0 s median** (p10 1.1 s, p90 3.0 s; 400 transactions) after matching: orders and cancels are off-chain (EIP-712 signatures),
  only the result goes on-chain.
- So **the Polygon mempool gives you no "extra second"**: the transaction appears there after the trade has happened and the book has moved.

## Quality of our own book recording

- The WebSocket recorder (`price_change`, `book`, `last_trade_price` events with transaction hash) captures **99.9% of trades** that
  later appear on-chain; 97.6% printed at the recorded best price, another 2.2% deeper in the book.
- A trap we fell into ourselves: the exchange can send `price_change` (level consumed) **before** the `last_trade_price` of the same
  trade. If you process events in arrival order, fills look "25% worse" than they really were.

## Gamma API: small things that eat a day

- `offset` pagination returns 422 somewhere after 2,100-2,500 records. A full crawl needs keyset pagination.
- `/events/keyset` ignores the `negRisk` filter (filter on the market-level field instead); `/markets/keyset` respects
  `end_date_min` / `end_date_max`.
- Without a `User-Agent` header some Gamma requests fail.

## Security: malicious "Polymarket bots"

Someone else's "Polymarket bot" code is one of the most common ways to lose a wallet:

- **StepSecurity, 2026-03-15**: [Malicious Polymarket Bot Hides in Hijacked dev-protocol GitHub Org and Steals Wallet Keys](https://www.stepsecurity.io/blog/malicious-polymarket-bot-hides-in-hijacked-dev-protocol-github-org-and-steals-wallet-keys).
  20+ fake repositories with inflated stars (`polymarket-trading-bot-new`, `polymarket-sports-trading-bot`,
  `polymarket-copy-trading-bot-sports` ...); dependencies include typosquatted npm packages `ts-bign` and `big-nunber` that steal
  private keys and `.env` files and open an SSH backdoor.
- **SafeDep, 2026-05-21**: [Polymarket npm Packages Steal Crypto Wallet Keys](https://safedep.io/malicious-polymarket-npm-crypto-wallet-drainer/).
  9 npm packages posing as Polymarket tools (`polymarket-bot`, `polymarket-copy-trading`, `polymarket-claude-code`,
  `polymarket-trading-cli`, `polymarket-terminal` ...) ask you to "paste your wallet key" and exfiltrate keys from environment
  variables and `.env` in plain text.

Do not install or run someone else's bot code on a machine that holds a funded wallet key without reading it first.
