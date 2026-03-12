# Forex Trading Reference

## Currency Pair Classification

### Majors (highest liquidity, tightest spreads)
- EUR/USD — Most traded pair globally; inversely correlated with DXY
- GBP/USD — "Cable"; volatile, sensitive to BOE/Fed divergence
- USD/JPY — "Gopher"; risk-on/off barometer; BOJ intervention risk
- USD/CHF — "Swissie"; safe haven flows
- USD/CAD — "Loonie"; Oil-correlated; watch WTI
- AUD/USD — "Aussie"; risk-on proxy; China/commodities exposure
- NZD/USD — "Kiwi"; similar to AUD; RBNZ rate sensitive

### Minors / Crosses (no USD)
- EUR/GBP, EUR/JPY, GBP/JPY ("Geppy" — volatile, wide stops needed)
- AUD/JPY, NZD/JPY — risk sentiment proxies
- EUR/CHF — SNB intervention history; cap removed 2015

### Exotics (wider spreads, lower liquidity)
- USD/ZAR, USD/NGN, USD/GHS, EUR/TRY
- Higher spreads mean wider stops and adjusted position sizing
- Avoid during illiquid hours

---

## Trading Sessions

| Session | Time (UTC) | Best Pairs |
|---|---|---|
| Sydney | 22:00–07:00 | AUD/USD, NZD/USD, AUD/JPY |
| Tokyo | 00:00–09:00 | USD/JPY, EUR/JPY, AUD/JPY |
| London | 07:00–16:00 | All majors; highest volume |
| New York | 12:00–21:00 | USD pairs; NFP at 13:30 UTC |
| **London-NY Overlap** | **12:00–16:00** | **Best for EUR/USD, GBP/USD** |
| Dead Zone | 21:00–00:00 | Avoid — thin liquidity, erratic |

**Key principle**: Volume drives volatility. Trade during session overlaps for cleanest price action.

---

## Pip Values (Standard Lot = 100,000 units)

| Pair | Pip Value (USD) |
|---|---|
| EUR/USD | $10 per pip |
| GBP/USD | $10 per pip |
| USD/JPY | ~$9.00 per pip |
| USD/CAD | ~$7.50 per pip |
| AUD/USD | $10 per pip |
| XAU/USD (Gold) | $10 per pip (1 pip = $0.10 move on 0.01 lot) |

**Position Size Formula**:
```
Lots = (Account × Risk%) / (SL_pips × PipValue)
Example: $10,000 × 1% / (30 pips × $10) = 0.33 lots
```

---

## Key Forex Concepts

### Carry Trade
- Borrow low-yield currency (JPY, CHF) → Buy high-yield currency (AUD, NZD, EM)
- Profitable in low-volatility, risk-on environments
- Unwinds violently during risk-off events (flash crashes in JPY pairs)

### Correlation Matrix (approximate)
```
EUR/USD ↔ GBP/USD: +0.85 (strong positive)
EUR/USD ↔ USD/CHF: -0.90 (strong negative)
EUR/USD ↔ USD/JPY: -0.40 (moderate negative)
AUD/USD ↔ NZD/USD: +0.88 (strong positive)
USD/CAD ↔ Oil:     -0.70 (negative — CAD strengthens with oil)
```

### DXY (Dollar Index) Impact
- DXY up → EUR/USD, GBP/USD, AUD/USD, Gold DOWN
- DXY down → Majors up, Gold up, EM currencies up
- Components: EUR (57.6%), JPY (13.6%), GBP (11.9%), CAD (9.1%), SEK (4.2%), CHF (3.6%)

---

## Central Banks & Divergence Trading

| Currency | Central Bank | Key Meeting Frequency |
|---|---|---|
| USD | Federal Reserve (Fed/FOMC) | 8x/year |
| EUR | European Central Bank (ECB) | 8x/year |
| GBP | Bank of England (BOE) | 8x/year |
| JPY | Bank of Japan (BOJ) | 8x/year |
| AUD | Reserve Bank of Australia (RBA) | 11x/year |
| NZD | Reserve Bank of New Zealand (RBNZ) | 8x/year |
| CAD | Bank of Canada (BOC) | 8x/year |
| CHF | Swiss National Bank (SNB) | 4x/year |

**Divergence trade**: When two central banks are at opposite ends of their rate cycle (one hiking, one cutting), the currency of the hiking bank typically outperforms.

---

## High-Impact Economic Releases

### USD (Most market-moving)
- **NFP (Non-Farm Payrolls)**: 1st Friday of month, 13:30 UTC — biggest volatility event
- **CPI**: Mid-month — drives Fed expectations
- **FOMC Statement + Press Conference**: Rate decision + dot plot + Powell's tone
- **GDP**: Quarterly advance estimate
- **ISM PMI**: Manufacturing (1st biz day) and Services (3rd biz day)
- **JOLTS, ADP, Retail Sales, PCE** — secondary but market-moving

### EUR
- **ECB Rate Decision + Lagarde presser**
- **Eurozone CPI Flash Estimate**
- **German Ifo, ZEW surveys**

### GBP
- **BOE MPC decision + minutes**
- **UK CPI, UK GDP, UK Employment**

### JPY
- **BOJ decision** — watch for YCC policy changes and intervention hints
- **Tokyo CPI** (leads national CPI)

---

## ICT / Smart Money Concepts (SMC) Reference

These concepts are commonly used in modern retail trading strategies:

**Liquidity**: Price hunts stop-loss clusters above highs (BSL) or below lows (SSL) before reversing
**Order Block (OB)**: Last bearish candle before a bullish move (bullish OB) or vice versa
**Fair Value Gap (FVG)**: 3-candle pattern where middle candle creates a gap; price tends to fill
**Break of Structure (BoS)**: Confirms trend continuation
**Change of Character (CHoCH)**: First sign of trend reversal
**Killzones** (high-probability entry windows):
- London Open: 07:00–09:00 UTC
- New York Open: 12:00–14:00 UTC
- London Close: 14:00–16:00 UTC
- Asian Range: 20:00–00:00 UTC (range definition, not breakout)

---

## Forex EA / Automated Trading Notes

When designing or evaluating Expert Advisors for forex:

**Session Filters** (MQL5):
```mql5
// Filter for London-NY overlap only
datetime currentTime = TimeCurrent();
MqlDateTime timeStruct;
TimeToStruct(currentTime, timeStruct);
int hour = timeStruct.hour;
bool inSession = (hour >= 12 && hour < 16); // UTC
```

**News Filter**: Use a news API or hard-coded high-impact event avoidance (±30 min)

**Spread Filter**: Always check current spread vs max allowed spread
```mql5
double currentSpread = SymbolInfoInteger(_Symbol, SYMBOL_SPREAD) * _Point;
if(currentSpread > maxSpreadAllowed) return; // Skip trade
```

**Recommended Backtesting Parameters**:
- Min 3 years of data, preferably 5+
- Use tick data or 1-min OHLC for M5/M15 strategies
- Out-of-sample period: last 6-12 months
- Target: Profit Factor >1.5, Sharpe >1.0, Max DD <15%, Win Rate >45%

---

## Common Forex Mistakes to Flag

1. **Overleveraging**: Never exceed 2% risk per trade
2. **Revenge trading**: EA logic must have max daily loss cutoff
3. **Ignoring spread on scalping**: A 3-pip spread destroys 2-pip TP strategies
4. **Not accounting for swap/rollover**: Overnight positions incur swap costs
5. **Correlation overexposure**: Running EUR/USD + GBP/USD + AUD/USD = essentially one trade
6. **Backtesting on insufficient data or wrong spread settings**: Always use realistic spreads
