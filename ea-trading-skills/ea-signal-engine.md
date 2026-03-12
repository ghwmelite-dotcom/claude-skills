---
name: ea-signal-engine
description: >
  Expert signal generation skill for MQL5 EAs. Use for ALL trade entry/exit logic including:
  SMC/ICT concepts (Order Blocks, Fair Value Gaps, CHoCH, BoS, liquidity sweeps, killzones),
  multi-timeframe analysis, technical indicator implementation (ATR, EMA, RSI, MACD, Bollinger,
  Stochastic, ADX), candlestick pattern detection, divergence logic, breakout strategies,
  mean reversion entries, trend-following entries, and translating any strategy concept into
  MQL5 signal code. Trigger on: "entry logic", "signal", "Order Block", "FVG", "CHoCH",
  "SMC", "ICT", "indicator", "pattern detection", "buy signal", "sell signal", "confluence".
  Always pair with ea-market-regime to filter signals by market state.
---

# EA Signal Engine Skill

Generates high-probability, multi-confluence entry and exit signals in MQL5.
Every signal must have minimum 2 confluence factors before triggering.

## Quick Reference Map

| Signal Type | Reference File |
|---|---|
| SMC/ICT concepts (OB, FVG, CHoCH, liquidity) | `references/smc-signals.md` |
| Technical indicators (ATR, EMA, RSI, ADX etc) | `references/indicators.md` |
| Candlestick patterns (Engulfing, Pin Bar etc) | `references/candle-patterns.md` |
| Multi-timeframe analysis & confluence | `references/mtf-confluence.md` |
| Asset-specific signal tuning | `references/asset-tuning.md` |

---

## Signal Architecture

### The STradeSignal Struct (used across all modules)
```mql5
enum SIGNAL_TYPE { SIGNAL_NONE, SIGNAL_BUY, SIGNAL_SELL };

struct STradeSignal {
   SIGNAL_TYPE   type;           // BUY / SELL / NONE
   double        entryPrice;     // Limit or market entry
   double        stopLoss;       // Absolute SL price
   double        takeProfit;     // Absolute TP price
   double        slDistance;     // SL distance in points (for lot sizing)
   double        rrRatio;        // Calculated R:R
   int           confluenceScore;// Number of confirming factors (min 2)
   string        reason;         // Log string: "OB+FVG+Session"
   datetime      signalTime;     // Bar open time when signal fired
};
```

### Signal Generation Flow
```
1. HTF Bias Check     (D1/H4 direction)
2. Liquidity Sweep    (BSL or SSL taken)
3. CHoCH Confirmed    (M15 structure shift)
4. OB/FVG Identified  (entry zone defined)
5. Session Active     (killzone timing)
6. Confluence Score   (≥2 = valid signal)
7. STradeSignal built and returned
```

---

## Core Signal: SMC London Open Gold (Primary)

```mql5
STradeSignal CSignalEngine::GetGoldLondonSignal() {
   STradeSignal sig;
   sig.type = SIGNAL_NONE;
   sig.confluenceScore = 0;

   // --- Factor 1: HTF Bias (H4) ---
   double h4_ema50 = iMA(_Symbol, PERIOD_H4, 50, 0, MODE_EMA, PRICE_CLOSE, 1);
   double h4_close = iClose(_Symbol, PERIOD_H4, 1);
   bool htfBullish = (h4_close > h4_ema50);
   bool htfBearish = (h4_close < h4_ema50);
   if(htfBullish || htfBearish) sig.confluenceScore++;

   // --- Factor 2: Asian Range (define 20:00-00:00 UTC) ---
   double asianHigh = GetAsianRangeHigh();  // See references/smc-signals.md
   double asianLow  = GetAsianRangeLow();

   // --- Factor 3: Liquidity Sweep Detection ---
   double m15_high = iHigh(_Symbol, PERIOD_M15, 1);
   double m15_low  = iLow(_Symbol, PERIOD_M15, 1);
   bool bslSwept = (m15_high > asianHigh);  // Buy-side liquidity taken
   bool sslSwept = (m15_low  < asianLow);   // Sell-side liquidity taken

   if(!bslSwept && !sslSwept) return sig;   // No sweep = no trade
   sig.confluenceScore++;

   // --- Factor 4: CHoCH on M15 ---
   bool choch = DetectCHoCH(PERIOD_M15, bslSwept ? MODE_BEARISH : MODE_BULLISH);
   if(!choch) return sig;
   sig.confluenceScore++;

   // --- Factor 5: FVG in retracement ---
   SFairValueGap fvg = FindFVG(PERIOD_M15, bslSwept ? FVG_BEARISH : FVG_BULLISH, 3);
   if(!fvg.isValid) return sig;
   sig.confluenceScore++;

   // --- Minimum confluence check ---
   if(sig.confluenceScore < 2) return sig;

   // --- Build signal ---
   double atr = iATR(_Symbol, PERIOD_H1, 14, 1);
   if(bslSwept && htfBearish) {
      sig.type       = SIGNAL_SELL;
      sig.entryPrice = fvg.high;               // Sell from FVG top
      sig.stopLoss   = fvg.high + atr * 1.5;
      sig.takeProfit = fvg.high - atr * 3.0;
   } else if(sslSwept && htfBullish) {
      sig.type       = SIGNAL_BUY;
      sig.entryPrice = fvg.low;                // Buy from FVG bottom
      sig.stopLoss   = fvg.low - atr * 1.5;
      sig.takeProfit = fvg.low + atr * 3.0;
   }
   sig.slDistance  = MathAbs(sig.entryPrice - sig.stopLoss) / _Point;
   sig.rrRatio     = MathAbs(sig.takeProfit - sig.entryPrice) /
                     MathAbs(sig.stopLoss   - sig.entryPrice);
   sig.reason      = "LiqSweep+CHoCH+FVG+HTFBias";
   sig.signalTime  = TimeCurrent();
   return sig;
}
```

---

## CHoCH Detection Function

```mql5
bool DetectCHoCH(ENUM_TIMEFRAMES tf, int expectedDirection) {
   // Track last 3 swing points
   double swingHigh[3], swingLow[3];
   int swingHighBar[3], swingLowBar[3];
   int swingCount = 0;

   for(int i = 2; i < 50 && swingCount < 3; i++) {
      double hi = iHigh(_Symbol, tf, i);
      double lo = iLow(_Symbol, tf, i);
      bool isSwingHigh = (hi > iHigh(_Symbol, tf, i-1) &&
                          hi > iHigh(_Symbol, tf, i+1));
      bool isSwingLow  = (lo < iLow(_Symbol, tf, i-1) &&
                          lo < iLow(_Symbol, tf, i+1));
      // Store swing points... (simplified logic)
      if(isSwingHigh || isSwingLow) swingCount++;
   }

   // Bullish CHoCH: price makes a higher high after a series of lower highs
   // Bearish CHoCH: price makes a lower low after a series of higher lows
   // Full implementation in references/smc-signals.md
   return false; // placeholder
}
```

---

## FVG (Fair Value Gap) Detection

```mql5
struct SFairValueGap {
   bool   isValid;
   double high;
   double low;
   double mid;
   int    barIndex;
};

SFairValueGap FindFVG(ENUM_TIMEFRAMES tf, int direction, int lookback) {
   SFairValueGap fvg;
   fvg.isValid = false;

   for(int i = 1; i < lookback; i++) {
      double c2_low  = iLow(_Symbol, tf, i+1);   // Candle 2 (oldest)
      double c2_high = iHigh(_Symbol, tf, i+1);
      double c1_low  = iLow(_Symbol, tf, i);     // Candle 1 (middle)
      double c1_high = iHigh(_Symbol, tf, i);
      double c0_low  = iLow(_Symbol, tf, i-1);   // Candle 0 (newest)
      double c0_high = iHigh(_Symbol, tf, i-1);

      // Bullish FVG: candle 0 low > candle 2 high (gap between them)
      if(direction == FVG_BULLISH && c0_low > c2_high) {
         fvg.isValid  = true;
         fvg.low      = c2_high;   // Bottom of gap
         fvg.high     = c0_low;    // Top of gap
         fvg.mid      = (fvg.high + fvg.low) / 2.0;
         fvg.barIndex = i;
         return fvg;
      }
      // Bearish FVG: candle 0 high < candle 2 low
      if(direction == FVG_BEARISH && c0_high < c2_low) {
         fvg.isValid  = true;
         fvg.high     = c2_low;    // Top of gap
         fvg.low      = c0_high;   // Bottom of gap
         fvg.mid      = (fvg.high + fvg.low) / 2.0;
         fvg.barIndex = i;
         return fvg;
      }
   }
   return fvg;
}
```

---

## Multi-Asset Signal Profiles

| Asset | Primary Signal | Entry Method | Key Filter |
|---|---|---|---|
| XAUUSD | London Open SMC | FVG limit order | Asian range sweep |
| GBPUSD | London-NY overlap breakout | OB market order | Session killzone |
| EURUSD | Trend continuation | EMA pullback | H4 trend direction |
| US500/NAS100 | Gap fill + VWAP | VWAP reclaim | Pre-market gap size |
| BTC/ETH | Halving cycle + OB | D1 OB limit | Funding rate filter |
| XAGUSD | Gold follower | Delayed gold signal | Gold/Silver ratio |

---

## Confluence Scoring System

Always calculate confluence score before executing:
```mql5
int CalculateConfluence(bool htfAligned, bool liquiditySwept,
                        bool chochConfirmed, bool inFVG,
                        bool inOrderBlock, bool sessionActive,
                        bool volumeConfirms, bool indicatorAligned) {
   int score = 0;
   if(htfAligned)        score += 2;  // HTF alignment = 2 points (most important)
   if(liquiditySwept)    score += 2;  // Liquidity sweep = 2 points
   if(chochConfirmed)    score += 2;  // CHoCH = 2 points
   if(inFVG)             score += 1;
   if(inOrderBlock)      score += 1;
   if(sessionActive)     score += 1;
   if(volumeConfirms)    score += 1;
   if(indicatorAligned)  score += 1;
   return score;
   // Minimum to trade: 4 points
   // High confidence:  7+ points
}
```

---

## Response Format for Signal Queries

When asked to build or review signal logic:
1. Show the full signal struct used
2. List each confluence factor explicitly
3. Show entry, SL, and TP calculation code
4. Flag any missing filters (session, spread, regime)
5. Suggest alternative confluence factors if score is weak
