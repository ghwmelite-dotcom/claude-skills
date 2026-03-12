---
name: ea-market-regime
description: >
  Market regime detection and filtering skill for MQL5 EAs. Use for ALL logic that determines
  WHAT kind of market conditions exist before generating signals: trend vs range detection,
  volatility regime (high/low/normal), session detection (London/NY/Asian/dead zone), news
  event filtering, spread spike detection, weekend/holiday detection, and any "should I be
  trading right now?" question. Trigger on: "market regime", "trending vs ranging", "session
  filter", "news filter", "volatility filter", "spread filter", "market conditions", "when
  to trade", "ATR regime", "ADX trend strength", "Bollinger squeeze", "dead zone", "killzone",
  "economic calendar". This is the EA's intelligence layer — signals without regime awareness lose money.
---

# EA Market Regime Skill

The regime filter determines WHETHER to trade. Signals tell you WHAT to trade.
Never generate signals without first passing through the regime filter.

## Quick Reference Map

| Topic | Reference File |
|---|---|
| Trend/Range detection algorithms | `references/trend-range.md` |
| Volatility regime classification | `references/volatility-regime.md` |
| Session timing & killzones | `references/sessions.md` |
| News event filter implementation | `references/news-filter.md` |
| Spread & liquidity monitoring | `references/spread-filter.md` |

---

## Regime Classification Enum

```mql5
enum ENUM_REGIME {
   REGIME_UNDEFINED = 0,   // Cannot determine — do not trade
   REGIME_TRENDING_BULL,   // Clear uptrend — long bias only
   REGIME_TRENDING_BEAR,   // Clear downtrend — short bias only
   REGIME_RANGING,         // Horizontal range — mean reversion
   REGIME_VOLATILE,        // High volatility — reduce size or skip
   REGIME_DEAD,            // Dead zone / thin liquidity — skip
};
```

---

## CRegimeFilter Class

```mql5
class CRegimeFilter : public IModule {
private:
   int    m_adxPeriod;
   double m_adxTrendThreshold;
   double m_adxRangeThreshold;
   int    m_atrPeriod;
   double m_volatilityMultiplier;

public:
   CRegimeFilter(int adxPeriod=14, double trendTH=25.0,
                 double rangeTH=20.0, int atrPeriod=14,
                 double volMult=2.0) {
      m_adxPeriod            = adxPeriod;
      m_adxTrendThreshold    = trendTH;
      m_adxRangeThreshold    = rangeTH;
      m_atrPeriod            = atrPeriod;
      m_volatilityMultiplier = volMult;
   }

   ENUM_REGIME GetCurrentRegime() {
      // Step 1: Check session
      if(IsDeadZone())     return REGIME_DEAD;

      // Step 2: Check volatility
      if(IsHighVolatility()) return REGIME_VOLATILE;

      // Step 3: Check trend/range
      double adx    = GetADX(PERIOD_H4);
      double plusDI = GetPlusDI(PERIOD_H4);
      double minusDI= GetMinusDI(PERIOD_H4);

      if(adx >= m_adxTrendThreshold) {
         if(plusDI > minusDI)  return REGIME_TRENDING_BULL;
         if(minusDI > plusDI)  return REGIME_TRENDING_BEAR;
      }
      if(adx < m_adxRangeThreshold) return REGIME_RANGING;

      return REGIME_UNDEFINED;  // Transitional — skip
   }

   bool IsDeadZone() {
      MqlDateTime t;
      TimeToStruct(TimeGMT(), t);
      int h = t.hour;
      // Dead zone: 21:00–00:00 UTC (thin liquidity)
      return (h >= 21 || h < 0);
   }

   bool IsHighVolatility() {
      double currentATR = iATR(_Symbol, PERIOD_H1, m_atrPeriod, 1);
      double avgATR     = 0;
      for(int i = 1; i <= 20; i++) avgATR += iATR(_Symbol, PERIOD_H1, m_atrPeriod, i);
      avgATR /= 20.0;
      return (currentATR > avgATR * m_volatilityMultiplier);
   }

   bool   IsReady()    { return true; }
   void   Reset()      { }
   string GetStatus()  {
      ENUM_REGIME r = GetCurrentRegime();
      string regimeStr[] = {"UNDEFINED","BULL_TREND","BEAR_TREND","RANGING","VOLATILE","DEAD"};
      return "Regime: " + regimeStr[r];
   }
};
```

---

## Session Filter (Full Implementation)

```mql5
class CSessionFilter {
private:
   int  m_gmtOffset;
   bool m_london, m_ny, m_asian;

public:
   CSessionFilter(int gmtOffset, bool london=true, bool ny=true, bool asian=false) {
      m_gmtOffset = gmtOffset;
      m_london = london;
      m_ny     = ny;
      m_asian  = asian;
   }

   bool IsActiveSession() {
      MqlDateTime t;
      TimeToStruct(TimeGMT(), t);
      int hour = t.hour;

      // London killzone: 07:00–09:00 UTC
      if(m_london && hour >= 7 && hour < 9)   return true;
      // London-NY overlap: 12:00–16:00 UTC
      if(m_ny     && hour >= 12 && hour < 16) return true;
      // NY close: 19:00–21:00 UTC
      if(m_ny     && hour >= 19 && hour < 21) return true;
      // Asian session: 00:00–02:00 UTC
      if(m_asian  && hour >= 0  && hour < 2)  return true;

      return false;
   }

   bool IsLondonOpen()   { /* 07:00-09:00 UTC */ return IsHourIn(7, 9); }
   bool IsNYOpen()       { /* 12:00-14:00 UTC */ return IsHourIn(12,14);}
   bool IsLondonNYOverlap() { return IsHourIn(12, 16); }

   bool IsWeekend() {
      MqlDateTime t;
      TimeToStruct(TimeCurrent(), t);
      return (t.day_of_week == 0 || t.day_of_week == 6);
   }

   bool IsPreWeekendClose() {
      // Friday 20:00+ UTC — close all positions
      MqlDateTime t;
      TimeToStruct(TimeGMT(), t);
      return (t.day_of_week == 5 && t.hour >= 20);
   }

private:
   bool IsHourIn(int from, int to) {
      MqlDateTime t;
      TimeToStruct(TimeGMT(), t);
      return (t.hour >= from && t.hour < to);
   }
};
```

---

## News Filter (Hardcoded High-Impact Events)

```mql5
class CNewsFilter {
private:
   int m_bufferMinutes;  // Minutes before/after to avoid

public:
   CNewsFilter(int bufferMin = 30) { m_bufferMinutes = bufferMin; }

   // Returns true if it's safe to trade (no news upcoming/recent)
   bool IsSafeToTrade() {
      // For live trading: integrate with ForexFactory API or DailyFX calendar
      // For offline safety: use time-of-day heuristics for known recurring events

      MqlDateTime t;
      TimeToStruct(TimeGMT(), t);
      int dayOfWeek = t.day_of_week;
      int hour      = t.hour;
      int minute    = t.min;
      int totalMin  = hour * 60 + minute;

      // NFP: 1st Friday, 13:30 UTC → block 13:00–14:00
      if(dayOfWeek == 5 && totalMin >= 780 && totalMin <= 840) return false;

      // FOMC: 8x per year at 19:00 UTC → runtime detection needed
      // Block 18:30–20:30 UTC on scheduled FOMC days (hardcode dates)

      // CPI: mid-month, 13:30 UTC
      if(totalMin >= 780 && totalMin <= 825) {
         // Additional date-based check needed for CPI days
      }

      return true;
   }
};
```

---

## Spread Monitor

```mql5
bool IsSpreadAcceptable(double maxSpreadPoints) {
   double currentSpread = (double)SymbolInfoInteger(_Symbol, SYMBOL_SPREAD);
   // Asset-specific max spread defaults:
   // XAUUSD: 50 points | EURUSD: 15 | GBPUSD: 20 | US500: 30 | BTC: 200
   return (currentSpread <= maxSpreadPoints);
}
```

---

## Regime-to-Strategy Mapping

```
REGIME_TRENDING_BULL  → Use: Trend-following signals (OB pullback longs, EMA bounce)
                        Skip: Mean reversion shorts
REGIME_TRENDING_BEAR  → Use: Breakdown signals, FVG fill shorts
                        Skip: Mean reversion longs
REGIME_RANGING        → Use: Range boundaries, RSI extremes, BBand bounce
                        Skip: Breakout signals
REGIME_VOLATILE       → Use: Reduce lot size by 50%, widen SL by 50%
                        Skip: Tight scalping signals
REGIME_DEAD           → Skip all signals. Period.
REGIME_UNDEFINED      → Skip all signals. Wait for clarity.
```

---

## Response Format for Regime Queries

When asked about market conditions or filters:
1. Show the ENUM_REGIME classification system
2. Provide the detection algorithm for the specific condition
3. Show how the regime plugs into OnTick() gating logic
4. Give asset-specific thresholds (gold vs forex vs indices)
5. Always flag weekend, news, and spread conditions
