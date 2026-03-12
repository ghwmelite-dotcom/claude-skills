# Indices Reference

## Major Global Indices

### US Indices (Most Liquid)

| Index | Ticker | Futures | Components | Character |
|---|---|---|---|---|
| S&P 500 | SPX | ES (CME) | 500 large-cap US | Benchmark; broad economy |
| Nasdaq 100 | NDX | NQ (CME) | 100 non-fin NASDAQ | Tech-heavy; growth-sensitive |
| Dow Jones | DJIA | YM (CME) | 30 price-weighted bluechips | Industrial bellwether |
| Russell 2000 | RUT | RTY (CME) | 2000 small-caps | Risk appetite; domestic economy |
| VIX | VIX | VX (CBOE) | Volatility index | Fear gauge; inverse to SPX |

**MT5 Instruments**: US500/SP500, US100/NAS100, US30/DJ30 (broker-dependent naming)

**Key Characteristics**:
- NDX is ~3x more volatile than SPX (beta ~1.3-1.5 vs SPX)
- DJIA price-weighted (Boeing $400 vs Apple $180 = Boeing has more weight despite smaller mktcap)
- Small-caps (RUT) lead bull markets up and bear markets down
- VIX >30 = high fear; VIX <15 = complacency; VIX >80 = extreme panic (COVID, GFC)

---

### European Indices

| Index | Country | Key Sectors |
|---|---|---|
| DAX 40 (DE40) | Germany | Autos, chemicals, industrials, financials |
| FTSE 100 (UK100) | UK | Energy, mining, financials (defensive, GBP-hedged) |
| CAC 40 (FRA40) | France | Luxury goods (LVMH), energy, banks |
| Euro Stoxx 50 | Eurozone | Pan-European bluechips |
| IBEX 35 | Spain | Banks, telecom, energy |

**Notes**:
- DAX = Total return index (dividends included) unlike most other indices
- FTSE 100 inversely correlated with GBP (weakening GBP = FTSE up due to USD revenues)
- European indices follow US closely but open 6 hours before NY

---

### Asian/Pacific Indices

| Index | Country | Notes |
|---|---|---|
| Nikkei 225 (JPN225) | Japan | Export-driven; JPY weakness → Nikkei up |
| Hang Seng (HK50) | Hong Kong | China proxy; volatile; tech and property exposure |
| ASX 200 (AUS200) | Australia | Mining, banks, REITs |
| Kospi (KOR200) | South Korea | Samsung, chips; China trade exposure |
| CSI 300 | China | Domestic A-shares; policy-driven |

---

## Indices Key Drivers

### Macroeconomic Factors
- **Interest Rates**: Rising rates = P/E compression = indices down (especially NDX)
  - Rule: 10Y yield and NDX have strong inverse relationship
- **Earnings**: S&P 500 EPS growth is the fundamental anchor
  - Earnings season (4x/year) = stock-specific volatility
- **USD**: Strong USD = headwind for multi-national earnings (SPX negative)
- **Inflation**: Moderate inflation OK; high inflation = margin compression = indices down
- **GDP Growth**: Positive growth expectations = risk-on = indices up
- **Unemployment**: Low unemployment = consumer spending = indices up (but also inflation fear)

### Market Sentiment / Technical Drivers
- **Breadth**: A/D line (advancers vs decliners); market narrow = bearish divergence
- **Sector rotation**: Defensive rotation (utilities, healthcare) = risk-off signal
- **Buybacks**: Major support for US indices; S&P 500 buybacks ~$1T/year
- **Options expiry (OpEx)**: Monthly and quarterly OPEX causes volatility clustering
  - Monthly OpEx: 3rd Friday of month — watch for pinning/magnets around strikes
  - Quarterly OpEx: March, June, September, December OpEx = "Quadruple Witching"

---

## VIX (Volatility Index) — The Fear Gauge

### Interpretation
| VIX Level | Market State |
|---|---|
| <12 | Extreme complacency; watch for reversal |
| 12-15 | Normal calm bull market |
| 15-20 | Slightly elevated; uncertain |
| 20-30 | Fear increasing; choppy |
| 30-40 | High fear; volatility spike |
| >40 | Extreme fear; capitulation possible (often near bottoms) |
| >50 | Crisis level (COVID: 85, GFC: 89) |

### VIX Trading Rules
- **VIX and SPX are typically inversely correlated** (-0.7 to -0.9)
- **VIX mean-reversion**: High VIX spikes typically revert; SPX bounces follow
- **VIX futures term structure**: Contango (normal) = front <back; Backwardation = fear
- **VVIX** (vol of vol): High VVIX = vol itself is uncertain = extreme conditions

---

## Sector Analysis & Rotation (S&P 500)

### 11 GICS Sectors
| Sector | Symbol | Character |
|---|---|---|
| Technology (IT) | XLK | Growth; rate-sensitive; largest weight |
| Healthcare | XLV | Defensive; M&A driven |
| Financials | XLF | Rate-sensitive (banks benefit from rising rates) |
| Consumer Discretionary | XLY | Cyclical; Amazon, Tesla heavy |
| Consumer Staples | XLP | Defensive; dividend payers |
| Energy | XLE | Oil-correlated; cyclical |
| Industrials | XLI | Economic cycle proxy; capital goods |
| Materials | XLB | Commodity-linked; cyclical |
| Utilities | XLU | Defensive; rate-sensitive (dividend alternative) |
| Real Estate (REITs) | XLRE | Rate-sensitive; income-oriented |
| Communication Services | XLC | Meta, Alphabet; ad revenue sensitive |

### Rotation Clock (Economic Cycle)
```
Recovery → Expansion → Peak → Contraction

Recovery:    Financials, Consumer Discretionary lead
Expansion:   Technology, Industrials, Materials outperform
Peak:        Energy, Materials, Healthcare rotate in
Contraction: Utilities, Healthcare, Consumer Staples outperform (defensives)
```

---

## Indices Trading Strategies

### Gap Fill Strategy
- US indices often gap open vs prior close
- Gaps have ~65% fill rate historically within same session
- Trade: Fade gap at open with SL beyond gap size; TP at prior close

### VWAP Reclaim / Rejection
- VWAP = Volume Weighted Average Price; institutional anchor
- Price above VWAP = buyers in control; below = sellers
- Entry: Wait for reclaim/rejection confirmation with volume

### Pre-Market and News-Driven Plays
- **Futures pre-market**: ES/NQ futures trade 23 hours; gaps visible before open
- **Fed-day playbook**: Initial move often reverses; wait for first 30-60 min before entering
- **Earnings**: Never hold over earnings without defined risk; straddle for volatility play

### Mean Reversion vs Trend
- Indices are better trend instruments on W1/D1 timeframes
- Intraday (M5-H1): Mean reversion works better in range-bound conditions
- Key: Identify regime first — trending (higher highs/lows) vs ranging (horizontal S/R)

---

## Indices in MT5 / EA Development Notes

```mql5
// Common MT5 index symbols (broker-dependent):
// S&P 500: US500, SP500, SPX500
// Nasdaq: US100, NAS100, NASDAQ
// Dow Jones: US30, DJ30, DOWJONES
// DAX: GER40, DE40, DAX40
// FTSE: UK100, FTSE100

// Note: Indices are CFDs on MT5 — no physical delivery
// Commission-based pricing; spreads widen massively outside hours
// Always check contract specs: lot size, tick value, min size

double tickValue = SymbolInfoDouble(_Symbol, SYMBOL_TRADE_TICK_VALUE);
double tickSize = SymbolInfoDouble(_Symbol, SYMBOL_TRADE_TICK_SIZE);
// Position sizing for indices uses same formula as forex
```

**Key difference from Forex**: Indices close (weekends, holidays) — gaps are common!
Always use reduced lot sizes around market open after weekend/holiday.
