# General Finance Reference

## Macroeconomics Framework

### The Interest Rate Transmission Mechanism
```
Central Bank Rate Change →
  → Bank lending rates change →
    → Consumer borrowing costs change →
      → Consumer spending changes →
        → Business revenues change →
          → Employment changes →
            → Inflation changes →
              → Central bank reacts again
```

**Lag effect**: Rate changes take 12-18 months to fully transmit through economy.

### Inflation & Monetary Policy
- **CPI (Consumer Price Index)**: Basket of consumer goods; headline vs core
- **PCE (Personal Consumption Expenditures)**: Fed's preferred inflation measure
- **PPI (Producer Price Index)**: Upstream inflation; leads CPI
- **Real Rate = Nominal Rate - Inflation Rate**
  - Positive real rate = restrictive policy
  - Negative real rate = accommodative (financial repression)

**Fed Dual Mandate**: Maximum employment + Price stability (2% inflation target)

### Yield Curve Analysis
```
Normal: Short yields < Long yields (positive slope) → Economy healthy
Flat: Similar yields across maturities → Uncertainty
Inverted: Short > Long yields → Recession predictor (2Y > 10Y spread negative)

2Y-10Y inversion has preceded every US recession since 1980 by 6-24 months.
Key spread: US 10Y minus 2Y Treasury yield
```

---

## Corporate Finance & Valuation

### Fundamental Valuation Metrics
| Metric | Formula | Note |
|---|---|---|
| P/E Ratio | Price / EPS | High = growth priced in; Low = value or risk |
| Forward P/E | Price / Next 12M EPS | Uses analyst estimates |
| PEG Ratio | P/E / EPS Growth % | <1 = potentially undervalued |
| P/S Ratio | Price / Revenue per share | For unprofitable companies |
| P/B Ratio | Price / Book Value | Banks: P/B <1 may indicate distress |
| EV/EBITDA | Enterprise Value / EBITDA | Best for acquisitions; debt-neutral |
| DCF | Sum of discounted future cash flows | Most rigorous; most assumption-sensitive |

### S&P 500 Historical P/E Context
- Long-run average: ~16-17x
- 2024 level: ~22-24x (expensive vs history but justified by mega-cap growth)
- Shiller CAPE (10-yr cyclically adjusted): Long-run average ~17; 2024 ~30+

### Free Cash Flow (FCF) — Key Metric
```
FCF = Operating Cash Flow - Capital Expenditures
FCF Yield = FCF / Market Cap (higher = better value)
```

---

## Portfolio Theory

### Modern Portfolio Theory (MPT)
- **Efficient Frontier**: Optimal portfolios maximizing return for given risk
- **Diversification**: Combining uncorrelated assets reduces portfolio volatility
- **Beta**: Sensitivity to market; Beta >1 = amplified market moves
- **Alpha**: Excess return vs benchmark; positive alpha = outperformance

### Key Risk Metrics
```
Sharpe Ratio = (Return - Risk-Free Rate) / Standard Deviation
  > 1 = good; > 2 = very good; > 3 = excellent

Sortino Ratio = (Return - Risk-Free Rate) / Downside Deviation
  Better than Sharpe as it only penalizes bad volatility

Max Drawdown = (Peak - Trough) / Peak × 100%
  Key for EA evaluation; target <15% for prop firm accounts

Calmar Ratio = CAGR / Max Drawdown
  > 1 = acceptable; > 2 = strong

Information Ratio = (Portfolio Return - Benchmark) / Tracking Error
  Consistency of alpha generation
```

### Asset Allocation Frameworks
**60/40 Portfolio** (Classic): 60% stocks, 40% bonds
- Benefits from negative stock/bond correlation (pre-2022)
- Failed in 2022 (both fell with rate hikes) — showed correlation breakdown

**Risk Parity**: Equal risk contribution from each asset class
- Requires leverage on bonds to match equity risk contribution
- Bridgewater's All Weather popularized this

**Permanent Portfolio** (Harry Browne): 25% Stocks / 25% Bonds / 25% Gold / 25% Cash
- Designed to perform in any economic environment
- Conservative; lower max drawdown at cost of returns

**Core-Satellite**: Passive core (80%) + Active satellite (20%)
- Reduces costs while allowing tactical positioning

---

## Fixed Income Concepts

### Bond Basics
- **Par Value**: Face value (usually $1,000)
- **Coupon**: Interest rate paid on par
- **Yield**: Return based on current price (inverse to price)
- **Duration**: Sensitivity to interest rate changes (in years)
- **Convexity**: Non-linear price/yield relationship; positive convexity = beneficial

### Credit Ratings
| Moody's | S&P | Fitch | Category |
|---|---|---|---|
| Aaa | AAA | AAA | Prime Investment Grade |
| Aa | AA | AA | High Grade |
| A | A | A | Upper Medium |
| Baa | BBB | BBB | Lower Medium (lowest IG) |
| Ba | BB | BB | Non-Investment Grade / Junk |
| B | B | B | Speculative |
| Caa-C | CCC-D | CCC-D | Highly Speculative / Default |

**Investment Grade**: BBB-/Baa3 and above
**High Yield / Junk**: BB+/Ba1 and below
**Spread**: Yield above Treasury; higher spread = more credit risk priced in

---

## Economic Indicators Cheat Sheet

### Leading Indicators (predict future economy)
- ISM Manufacturing PMI (>50 = expansion)
- Conference Board Leading Index
- Yield curve (10Y-2Y spread)
- Building permits / Housing starts
- Stock market performance
- Consumer confidence

### Coincident Indicators (current state)
- GDP growth rate
- Non-Farm Payrolls
- Industrial production
- Personal income

### Lagging Indicators (confirm trends)
- Unemployment rate (lags economy by months)
- CPI inflation
- Bank lending rates
- Corporate profits

---

## Behavioral Finance — Patterns to Know

### Common Cognitive Biases in Trading
- **Loss Aversion**: Losses feel 2x worse than gains feel good → holding losers too long
- **Confirmation Bias**: Seeking info that confirms existing view → ignoring counter-arguments
- **Recency Bias**: Over-weighting recent events → buying tops, selling bottoms
- **Anchoring**: Over-relying on first price seen → poor entry/exit decisions
- **Disposition Effect**: Selling winners too early, holding losers too long
- **Overconfidence**: Overestimating skill after wins → overleveraging
- **FOMO (Fear of Missing Out)**: Chasing pumps → buying at tops

### Market Cycle Psychology (Stages)
```
Optimism → Excitement → Thrill → Euphoria (maximum risk!)
→ Anxiety → Denial → Panic → Capitulation
→ Despondency → Depression (maximum opportunity!)
→ Hope → Relief → Optimism again
```

---

## Financial Statement Analysis

### Three Core Statements
**Income Statement**: Revenue → COGS → Gross Profit → Operating Expenses → EBITDA → EBT → Net Income
**Balance Sheet**: Assets = Liabilities + Equity; assess leverage, liquidity
**Cash Flow Statement**: Operating + Investing + Financing activities; FCF = operating - capex

### Key Ratios by Category
```
LIQUIDITY:
  Current Ratio = Current Assets / Current Liabilities (>2 = healthy)
  Quick Ratio = (Current Assets - Inventory) / Current Liabilities (>1 = OK)

LEVERAGE:
  Debt/Equity = Total Debt / Shareholders' Equity (<2 = manageable)
  Interest Coverage = EBIT / Interest Expense (>3 = comfortable)

PROFITABILITY:
  Gross Margin = (Revenue - COGS) / Revenue
  EBITDA Margin = EBITDA / Revenue
  ROE = Net Income / Shareholder Equity (>15% = strong)
  ROA = Net Income / Total Assets (>5% = good)

EFFICIENCY:
  Asset Turnover = Revenue / Total Assets
  Inventory Turnover = COGS / Average Inventory
```

---

## Ghana/Africa Financial Context

### Ghana Specific
- **GSE (Ghana Stock Exchange)**: Limited liquidity; dominated by banks and telcos
- **GHS/USD**: Cedi historically depreciates; inflation-driven currency risk for GHS savings
- **Government Bonds (GOG)**: High yields (20%+) but GHS-denominated; inflation eats returns
- **Treasury Bills**: 91-day, 182-day, 364-day; liquid; government-backed
- **Bank of Ghana (BOG)**: Sets monetary policy rate; tracks inflation
- **Ghana DDEP (Domestic Debt Exchange)**: 2023 restructuring affected local bonds
- **Mobile Money**: GhIPSS and MoMo as payment infrastructure

### Investing from Ghana
- **Foreign exchange controls**: BOG regulations on FX outflows
- **Offshore brokers**: Interactive Brokers, XM, Exness, HotForex — popular for Ghanaian traders
- **Remittances**: Wise, WorldRemit for funding offshore accounts
- **Crypto as FX hedge**: Common practice given GHS depreciation history
- **FinTech landscape**: Zeepay, Paystack, Flutterwave operating in Ghana
