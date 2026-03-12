---
name: ea-backtest-lab
description: >
  Expert backtesting, optimization and performance analysis skill for MQL5 EAs. Use for ALL
  backtesting strategy: MT5 Strategy Tester configuration, walk-forward testing, Monte Carlo
  simulation, parameter optimization (genetic algorithm, grid search, exhaustive), overfitting
  detection, out-of-sample validation, interpreting backtest metrics (Profit Factor, Sharpe,
  Calmar, Max DD, Recovery Factor, Expected Payoff), optimization maps, custom optimization
  criteria, and "is this EA curve-fitted?" analysis. Trigger on: "backtest", "optimize",
  "walk-forward", "Monte Carlo", "Profit Factor", "Sharpe Ratio", "overfitting", "curve fitting",
  "out of sample", "strategy tester", "optimization criteria", "forward test", "robustness".
  A strategy untested is a strategy unproven — this skill validates everything.
---

# EA Backtest Lab Skill

Backtesting is not validation — it's hypothesis testing. This skill teaches you
to test correctly, interpret honestly, and avoid the traps that fool most traders.

## Quick Reference Map

| Topic | Reference File |
|---|---|
| MT5 Strategy Tester setup guide | `references/tester-setup.md` |
| Walk-forward testing methodology | `references/walk-forward.md` |
| Optimization theory & overfitting | `references/optimization.md` |
| Performance metrics interpretation | `references/metrics.md` |
| Monte Carlo simulation | `references/monte-carlo.md` |

---

## Metric Interpretation Framework

### Pass/Fail Scorecard
```
Metric              | Fail    | Acceptable | Good    | Excellent
--------------------|---------|------------|---------|----------
Profit Factor (PF)  | <1.2    | 1.2–1.4    | 1.4–1.7 | >1.7
Win Rate (WR)       | <35%    | 35–45%     | 45–55%  | >55%
Sharpe Ratio        | <0.5    | 0.5–0.9    | 0.9–1.5 | >1.5
Calmar Ratio        | <0.5    | 0.5–1.0    | 1.0–2.0 | >2.0
Max Drawdown        | >20%    | 15–20%     | 10–15%  | <10%
Recovery Factor     | <1.0    | 1.0–2.0    | 2.0–4.0 | >4.0
Expected Payoff     | <0      | 0–5        | 5–15    | >15 USD
Total Trades (3yr)  | <100    | 100–200    | 200–500 | >500
Consecutive Losses  | >10     | 8–10       | 5–7     | <5
```

### The 3 Metrics That Matter Most (in order)
1. **Profit Factor** — ratio of gross profit to gross loss. PF of 1.4+ means the strategy makes $1.40 for every $1.00 lost.
2. **Max Drawdown** — worst peak-to-trough loss. Must stay inside prop firm limits.
3. **Recovery Factor** — net profit / max drawdown. Shows how quickly losses are recovered.

### How to Read a Backtest Report
```
Net Profit:    Headline figure — adjust for realistic spread/commission
Gross Profit:  Total of winning trades
Gross Loss:    Total of losing trades
PF:            Gross Profit / Gross Loss (target >1.4)
Total Trades:  More = more statistically significant (>200 minimum)
WR:            Less meaningful alone — always combine with RR
Avg Win:       Should be > Avg Loss * (1 - WR) / WR for profitability
Max DD:        Must not exceed prop firm limit at ANY point
Recovery Factor: Net Profit / Max DD (target >2.0)
Sharpe:        Return / Volatility of returns (target >1.0)
Expected Payoff: Average profit per trade (must be positive after costs)
```

---

## Overfitting Detection Checklist

```
Signs your EA is curve-fitted (overfitted):

✗ Backtest PF > 3.0 but forward test PF < 1.2
✗ Strategy only works on very specific parameter set (sharp optimization peak)
✗ Performance drops significantly outside tested date range
✗ High WR (>70%) with very tight SL — often means stop hunt in backtester
✗ Optimization island — only one narrow parameter range works
✗ Fewer than 200 trades in 3-year backtest
✗ Strategy only works on one specific broker's feed
✗ Results change dramatically with different tick data quality

Green flags (robustness indicators):
✓ Stable PF across broad parameter ranges (flat optimization map)
✓ Out-of-sample period (last 12 months untouched) shows similar metrics
✓ Walk-forward test maintains PF > 80% of in-sample PF
✓ Monte Carlo 95th percentile drawdown < prop firm limit
✓ Works on multiple symbols with similar (not identical) parameters
✓ Performance consistent across different market regimes (2020, 2022, 2024)
```

---

## Walk-Forward Testing Protocol

```
Standard walk-forward methodology:

Total data:     5 years (2019–2024)
In-sample:      3 years (2019–2021) — optimize here
Out-of-sample:  1 year  (2022)      — validate here
Forward test:   1 year  (2023–2024) — live proxy

Walk-forward windows:
  Window size:  6 months in-sample + 2 months out-of-sample
  Step:         2 months forward each iteration
  Iterations:   18 windows across 5-year dataset

Pass criteria:
  WF Efficiency = (Avg OOS PF) / (Avg IS PF) > 0.7
  If WF efficiency < 0.7 → strategy is overfitted → do not trade live
```

---

## MT5 Strategy Tester Configuration

```
Mode:          Every Tick Based on Real Ticks (most accurate)
             OR Every Tick (faster; less accurate for scalpers)
Modeling:      Use tick data for M5 or lower strategies
               Use OHLC for H1+ strategies (acceptable)
Spread:        Use current spread OR set realistic fixed spread:
               XAUUSD: 25-35 | EURUSD: 8-12 | GBPUSD: 12-18 | US500: 30
Deposit:       Match actual account size ($10,000 for prop firm)
Leverage:      Match broker (1:100 typical for prop firm)
Optimization:  Genetic Algorithm (faster) for initial scan
               Exhaustive for final validation of best parameters

CRITICAL: Never use "Open Prices Only" for entry/exit accuracy
CRITICAL: Always include commission in results ($7 per lot round-trip typical)
CRITICAL: Run at least 3 years minimum, 5 years preferred
```

---

## Custom Optimization Criterion (MQL5)

```mql5
// In EA main file — override to use custom criterion in optimizer
double OnTester() {
   double pf            = TesterStatistics(STAT_PROFIT_FACTOR);
   double maxDD         = TesterStatistics(STAT_EQUITY_DD);
   double netProfit     = TesterStatistics(STAT_PROFIT);
   double totalTrades   = TesterStatistics(STAT_TRADES);
   double sharpe        = TesterStatistics(STAT_SHARPE_RATIO);
   double recoveryFactor= TesterStatistics(STAT_RECOVERY_FACTOR);

   // Reject results with too few trades
   if(totalTrades < 100) return -1.0;
   // Reject results with negative profit
   if(netProfit <= 0)    return -1.0;
   // Reject results with extreme DD
   if(maxDD > 15.0)      return -1.0;
   // Reject if PF below minimum
   if(pf < 1.3)          return -1.0;

   // Custom score: weighted combination
   double score = (pf * 2.0) +           // Weight PF highest
                  (sharpe * 1.5) +        // Sharpe important
                  (recoveryFactor * 1.0)  // Recovery factor
                  - (maxDD * 0.5);        // Penalize drawdown

   return score;
}
```

---

## Monte Carlo Simulation Framework

```
Run 1,000 Monte Carlo simulations by:
1. Randomly shuffling the sequence of trades
2. Randomly skipping 10% of trades (simulate missed signals)
3. Randomly adjusting each trade result by ±20% (slippage simulation)

Key outputs:
- 95th percentile Max DD → must be < prop firm limit
- 5th percentile Net Profit → must still be positive
- Probability of ruin (DD > 10%) → must be < 5%

Tools: MT5 built-in Monte Carlo | Quant Analyzer | Excel simulation
```

---

## Response Format for Backtest Queries

1. Score the provided metrics against the pass/fail table
2. Identify the weakest metric and why it matters
3. Check for overfitting signs explicitly
4. Recommend next validation step (walk-forward, OOS, Monte Carlo)
5. Give specific parameter ranges to optimize next
6. State whether the EA is prop-firm ready based on the numbers
