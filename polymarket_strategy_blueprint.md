# Polymarket Inefficiency Trading Blueprint

## 1) High-level approach
Build a **strategy execution platform** with these layers:

1. **Data ingestion**
   - Market/order-book/trade data from Polymarket APIs.
   - Wallet/account activity feeds for selected “smart money” addresses.
   - Optional external signals (news, social sentiment, exchange prices, macro calendars).
2. **Feature + signal engine**
   - Converts raw data into metrics (mispricing, momentum, liquidity, spread, crowd skew, etc.).
3. **Strategy modules**
   - Independent strategy plugins (copy trading, mean reversion, event momentum, micro-scalping).
4. **Risk + portfolio layer**
   - Capital allocation, position limits, correlation caps, drawdown guardrails.
5. **Execution engine**
   - Smart order placement, slicing, slippage controls, maker/taker routing.
6. **Monitoring UI**
   - Deploy/stop strategies, budgets, live PnL, exposures, diagnostics.

---

## 2) UI concept (what to control)
For each strategy deployment, configure:

- **Budget** (e.g., $500/day, $5,000 total cap).
- **Max per market** (absolute and % of daily volume).
- **Max open positions**.
- **Allowed market categories** (politics, sports, crypto, macro).
- **Entry rules** and **exit rules**.
- **Time horizon** (scalper, intraday, swing, event-only).
- **Risk limits**:
  - max daily loss,
  - max drawdown,
  - max slippage,
  - min liquidity threshold.
- **Execution style**:
  - aggressive (cross spread quickly),
  - passive (resting orders),
  - hybrid.

UI panels:
- **Deployments dashboard** (running/stopped, budget used, current PnL).
- **Trade blotter** (fills, failed orders, cancel reason).
- **Market scanner** (inefficiency scores).
- **Strategy comparison** (Sharpe-like score, win rate, average hold time).

---

## 3) Candidate strategies

### A. Copy-trading “insider-like” wallets
- Track top wallets by:
  - long-term ROI,
  - hit rate,
  - edge persistence,
  - low overfitting behavior (not just one lucky event).
- Enter after a signal threshold (e.g., repeated buys over X minutes).
- Add anti-crowding filter: avoid too-late entries after major move.

**Edge hypothesis:** Some informed participants act early before broader repricing.

### B. Structural “NO bias” strategy
- Many long-tail YES contracts are overbought due to lottery psychology.
- Systematically evaluate when NO side has better expected value.
- Screen for poor resolution clarity and low-quality narratives.

**Edge hypothesis:** Retail overpays for exciting YES outcomes.

### C. Early-YES momentum (with strict exits)
- Opposite regime: markets with underreaction at launch.
- Enter small early YES in news-driven markets with likely attention ramp.
- Exit on first volatility spike / target return.

**Edge hypothesis:** Early information diffusion creates temporary trend.

### D. Microstructure scalping
- Capture spread/mean reversion in high-activity markets.
- Maker-biased orders with inventory limits.
- Requote around short-term fair-value model.

**Edge hypothesis:** Temporary order-book imbalances mean-revert.

### E. Cross-market consistency arb (statistical)
- Identify logically related markets with inconsistent probabilities.
- Trade basket to exploit incoherence and converge later.

**Edge hypothesis:** Fragmented attention causes cross-contract mispricing.

### F. Time-decay/event-clock strategy
- As resolution nears, weak-conviction markets can snap to fairer odds.
- Build calendar-aware position sizing and forced close windows.

**Edge hypothesis:** Price efficiency improves closer to settlement deadlines.

---

## 4) Decision intelligence ("bulk intelligence")
Use a **meta-model** that scores opportunities from all strategies:

- Inputs:
  - expected edge,
  - confidence,
  - liquidity quality,
  - estimated slippage,
  - strategy recent regime performance.
- Output:
  - rank opportunities,
  - allocate capital dynamically,
  - throttle or pause degraded strategies.

A simple first version can be rule-based; later use contextual bandits or Bayesian allocation.

---

## 5) Sniper vs. many tiny trades

### Many tiny trades
- Pros: diversified, smoother equity curve, less event risk concentration.
- Cons: execution overhead, fees/slippage accumulation.
- Best when: strong microstructure signals + good automation.

### Sniper trades
- Pros: higher conviction, lower operational noise.
- Cons: variance and headline/event risk.
- Best when: identifiable catalyst with clear mispricing.

**Practical hybrid:**
- Core allocation to tiny systematic trades.
- Satellite allocation to sniper setups with strict risk cap.

---

## 6) Risk framework (non-negotiable)
- Hard kill-switch if daily loss > threshold.
- Exposure caps by market/topic/event date.
- Liquidity-aware sizing (never exceed X% visible depth).
- Max adverse excursion alerts.
- Post-trade attribution (alpha vs slippage vs timing).

---

## 7) Suggested tech stack
- **Python** backend for data/scoring/execution logic.
- **FastAPI** for strategy control APIs.
- **PostgreSQL** for trades, markets, metrics history.
- **Redis** for low-latency queues/state.
- **React/Next.js** frontend for deployment UI.
- **Docker** + scheduled workers (cron/Celery/RQ).

---

## 8) Implementation roadmap (iterative)

### Phase 1: Foundation (1–2 weeks)
- Connect API + ingest market/trade/order-book snapshots.
- Create schema for markets, fills, positions, PnL.
- Build a minimal dashboard (read-only).

### Phase 2: Single strategy MVP (1–2 weeks)
- Implement one strategy (e.g., NO-bias or copy-trade).
- Add paper trading mode + backtest replay.
- Add deployment controls (budget, max position, stop).

### Phase 3: Multi-strategy allocator (2–4 weeks)
- Add 2–3 additional strategies.
- Add central risk engine and capital allocator.
- Add execution quality metrics.

### Phase 4: Optimization + scale
- Hyperparameter search by regime.
- Better wallet ranking and decay scoring.
- Advanced execution (order slicing and queue-position logic).

---

## 9) Key metrics to track from day one
- Net PnL, gross edge, fees, slippage.
- Win rate and payoff ratio.
- Max drawdown and time-to-recovery.
- Fill ratio and average execution delay.
- Strategy correlation and concentration risk.
- Edge decay after signal (how fast alpha disappears).

---

## 10) Questions to tailor your system
1. What starting capital and maximum acceptable daily drawdown do you want?
2. Are you targeting **fully automated execution** or semi-automated with human confirmation?
3. Which style do you prefer first: **many tiny trades**, **sniper**, or **hybrid**?
4. Which market categories are in/out of scope?
5. Do you want a **paper trading period** first (recommended), and how long?
6. What is your target cadence: number of trades/day and holding period?
7. Do you already have candidate wallets/accounts to monitor?
8. Should we prioritize “safe steady” Sharpe-like performance or higher-risk aggressive growth?


---

## 11) Tailored v1 configuration (based on your constraints)

### Capital and risk
- **Starting capital:** 500 USDT.
- **Daily max drawdown:** 5% (25 USDT).
- **Hard stop behavior:**
  - stop opening new positions at -3.5% daily PnL,
  - force close all positions at -5%,
  - cooldown until next UTC day.

### Execution model
- **Mode:** fully automated by default, with optional manual override toggles.
- **Operator agency controls:**
  - approve/reject high-risk trades,
  - pause any strategy,
  - adjust per-strategy budget live,
  - emergency flat-all kill switch.

### Trade frequency and style
- **Initial cap:** 5 trades/day total.
- **Preference:** prioritize high-conviction setups over high-frequency tiny trades.
- **Policy:** if expected value of one sniper setup beats basket EV of many small trades, allocate to sniper.

### Market scope
- No category restrictions; select opportunities by **risk-adjusted expected value** only.

### Validation period
- **Paper trading:** 1 week minimum.
- **Extension rule:** add 1–2 more weeks if variance is high or metrics unstable.

### Performance objective
- Priority order:
  1. Protect capital and maintain steady returns.
  2. Maximize daily profit subject to risk constraints.
  3. Gradually test aggressive overlays with small budget slices.

---

## 12) Strategy tournament framework ("competing strategies")
Run strategies as competitors with shared capital and dynamic ranking.

### Candidate v1 strategy set
1. **Safe baseline**: NO-bias + strict liquidity filters.
2. **Event sniper**: catalyst-driven entries, tighter stop rules.
3. **Wallet-following bootstrap**: begin with discovery mode until reliable wallets are identified.

### Capital allocation logic
- Start with:
  - 70% baseline safe strategy,
  - 20% event sniper,
  - 10% experimental.
- Re-rank daily by a blended score:
  - 40% recent risk-adjusted return,
  - 30% drawdown stability,
  - 20% execution quality (slippage/fill ratio),
  - 10% regime confidence.
- Auto-rebalance only within guardrails; never exceed daily loss limits.

### Promotion / demotion rules
- Promote strategy if it beats baseline score for 3 consecutive evaluation windows.
- Demote or pause strategy after 2 limit breaches (slippage spike, drawdown breach, or execution failures).

---

## 13) First concrete build iteration

### Week 1 (paper mode)
- Implement market ingestion + feature store.
- Build two live-paper strategies: baseline NO-bias and event sniper.
- Enforce account-level 5 trades/day and 5% drawdown stop.
- Add scoreboard UI: PnL, EV, drawdown, fill quality, strategy rank.

### Week 2 (decision checkpoint)
- If stable metrics: keep 1 more week and start tiny real allocation.
- If unstable metrics: extend paper test by 1–2 weeks and tune filters/position sizing.

### Go-live criteria
- Positive paper expectancy net of fees/slippage.
- Maximum observed paper drawdown within allowed 5% daily envelope.
- No critical execution failures for 5 consecutive days.
