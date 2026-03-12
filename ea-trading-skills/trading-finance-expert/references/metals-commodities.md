# Metals & Commodities Reference

## Precious Metals

### Gold (XAU/USD) — The Flagship
**Characteristics**:
- Quoted in USD per troy ounce; 1 troy oz = 31.1g
- Spot: XAU/USD | Futures: COMEX GC | ETF: GLD, IAU
- Average daily range: 15–30 USD; volatile on risk events (50–80 USD)
- Typical pip/tick: $0.10 per 0.01 lot; $10 per standard lot

**Gold Price Drivers**:
| Driver | Direction |
|---|---|
| USD strengthening (DXY up) | Gold DOWN |
| Real interest rates falling | Gold UP |
| Inflation rising (CPI) | Gold UP (store of value) |
| Geopolitical risk | Gold UP (safe haven) |
| Risk-off sentiment | Gold UP |
| Central bank buying | Gold UP (structural) |
| Fed rate hikes (real rates) | Gold DOWN |
| Equity market crash | Gold UP (flight to safety) |

**Key Correlations**:
- DXY: Strong negative (-0.80 typically)
- US 10Y Real Yield: Strong negative (most reliable)
- SPX: Mild negative to neutral; can diverge during crises
- Silver (XAG): Highly positive (+0.85); silver amplifies gold moves

**Structural Demand**:
- Central bank purchases (Russia, China, India buying aggressively post-2022)
- Jewelry (India, China dominant)
- Industrial (electronics, dentistry)
- Investment (ETFs, physical bars/coins)

**Technical Levels Reference**:
- Key psychological: $1,000, $1,500, $1,800, $2,000, $2,500, $3,000
- Support structure tends to form at previous ATH after breakout
- Weekly/monthly closes above round numbers = trend confirmation

**Trading Gold in MT5 / MQL5**:
```mql5
// Gold symbol varies by broker: XAUUSD, GOLD, XAUUSDm
// Typical contract: 100 oz per standard lot
// Min lot: 0.01 (1 oz at $0.10/pip)
// Spread: 20-50 points during liquid hours, 100+ during thin markets

// ATR-based SL for gold:
double atr = iATR(_Symbol, PERIOD_H1, 14, 1);
double slDistance = atr * 1.5; // 1.5x ATR stop
```

---

### Silver (XAG/USD)
- More volatile than gold; industrial demand adds cyclical component
- Gold/Silver Ratio: Historically 40-80; extreme readings mean-revert
  - High ratio (80+): Silver cheap relative to gold → buy silver
  - Low ratio (<40): Silver expensive → buy gold
- Industrial demand: Solar panels (major!), electronics, batteries
- Investment demand: More speculative; "poor man's gold"
- Typical daily range: 0.30–0.80 USD

### Platinum (XPT/USD)
- Primarily industrial: catalytic converters (diesel vehicles), hydrogen fuel cells
- Often trades at premium to gold historically; now at discount
- Platinum/Gold ratio: Currently inverted from historical norm
- South Africa dominant producer (>70% global supply) — supply disruption risk

### Palladium (XPD/USD)
- Catalytic converters (gasoline vehicles — opposite of platinum)
- Russia dominant supplier (~40%) — geopolitical premium
- EV transition = long-term demand headwind
- Very illiquid; wide spreads; institutional instrument

---

## Industrial Metals

### Copper (XCU/USD or HG Futures)
- **"Doctor Copper"**: Leading economic indicator
- Rising copper = economic growth expected; falling = recession risk
- China is #1 consumer (~50% global demand)
- LME (London) and COMEX (NY) are key exchanges
- Correlation: AUD/USD positive (Australia = major copper exporter)

### Iron Ore
- Steel production proxy; China demand driven
- Not directly tradeable on retail platforms; tracked via miners (BHP, RIO) or futures

---

## Energy Commodities

### Crude Oil (WTI and Brent)
- **WTI (CL)**: US crude; NYMEX; USD/barrel
- **Brent (EB/BRN)**: Global benchmark; ICE; ~$3-5 premium to WTI
- **OPEC+ decisions**: Most impactful regular event for oil
- **USD/CAD correlation**: Oil up → CAD strengthens → USD/CAD falls

### Natural Gas (NG)
- Highly seasonal; weather-driven
- European TTF vs US Henry Hub pricing divergence
- Storage reports (EIA weekly) = high volatility events

---

## Metals Trading Strategy Frameworks

### Gold Range Trading (During Consolidation)
```
Identify: HTF (D1/W1) range between key S/R
Entry: LTF (H1/H4) reversal signals at range extremes
TP: Opposite range boundary
SL: Beyond range by 1x ATR
Filter: Avoid during FOMC week / major data
```

### Gold Trend Following (During Trending Markets)
```
Trend identified on W1/D1 using 50 EMA
Entry: Pullbacks to D1 EMA or FVG zones on H4
SL: Below last swing low (uptrend) / above last swing high (downtrend)
TP: 1.5x-2x RR or prior ATH level
```

### Risk Event Positioning (Gold as Hedge)
```
Before FOMC: Reduce gold position by 50% (high uncertainty)
After hawkish surprise: Expect gold dump; wait for support confirmation before buying
After dovish surprise: Buy gold breakout confirmation
Geopolitical event: Gold spike — fade aggressively if not fundamentally driven
```

---

## Commodity Seasonality Patterns

| Commodity | Seasonal Tendency |
|---|---|
| Gold | Stronger Jan-Feb (Indian/Chinese demand) and Sep-Oct |
| Silver | Follows gold with amplification |
| Natural Gas | Strong Oct-Feb (heating season); weak Apr-Sep |
| Crude Oil | Strong Mar-Jun (driving season build) |
| Copper | Strong Q1 (China restocking post-Lunar New Year) |

---

## Key Resources & Data Sources

- **COMEX COT Report** (Commitment of Traders): Shows positioning of commercials vs speculators; extreme net long/short in non-commercials = contrarian signal
- **World Gold Council**: Central bank demand data, ETF flows
- **LME (London Metal Exchange)**: Industrial metals inventory levels
- **EIA Reports**: Oil/gas inventory data (weekly)
- **CFTC**: Futures positioning across all commodities
