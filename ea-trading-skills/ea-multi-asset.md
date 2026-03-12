---
name: ea-multi-asset
description: >
  Multi-asset and multi-pair EA management skill for MQL5. Use for ALL logic involving
  trading more than one instrument: portfolio-level risk management, per-symbol configuration,
  asset class parameters (forex pairs, gold/metals, indices, crypto CFDs), symbol manager
  class design, correlation-based position limits, running multiple strategies simultaneously,
  magic number management per symbol, asset-specific ATR/spread/session parameters, and
  any "how do I trade multiple pairs in one EA" question. Trigger on: "multi-pair", "portfolio EA",
  "multiple symbols", "all pairs", "forex and gold", "symbol manager", "asset class",
  "per-pair settings", "correlation management", "basket trading", "run on all pairs",
  "trade everything". This skill turns a single-pair EA into a full trading system.
---

# EA Multi-Asset Manager Skill

A true trading system trades multiple instruments simultaneously with proper
portfolio-level risk controls. This skill handles all the complexity of
running one brain across many markets.

## Quick Reference Map

| Topic | Reference File |
|---|---|
| Symbol configuration system | `references/symbol-config.md` |
| Portfolio risk & correlation | `references/portfolio-risk.md` |
| Asset class parameters | `references/asset-classes.md` |
| Multi-pair EA architecture | `references/multi-pair-arch.md` |
| Magic number & trade isolation | `references/magic-numbers.md` |

---

## Asset Class Parameter Profiles

```mql5
struct SSymbolConfig {
   string   symbol;
   int      magicNumber;
   double   maxSpreadPoints;    // Max allowed spread
   double   atrMultiplierSL;   // SL = ATR × this
   double   atrMultiplierTP;   // TP = ATR × this
   int      atrPeriod;         // ATR calculation period
   ENUM_TIMEFRAMES entryTF;    // Entry timeframe
   ENUM_TIMEFRAMES htfTF;      // Higher timeframe for bias
   int      sessionStartUTC;   // Session open hour (UTC)
   int      sessionEndUTC;     // Session close hour (UTC)
   double   correlationGroup;  // Group ID for correlation control
   string   assetClass;        // "forex", "metal", "index", "crypto"
};
```

### Pre-Built Asset Profiles

```mql5
SSymbolConfig g_SymbolProfiles[] = {
// FOREX MAJORS
// symbol    magic   spread  slMult tpMult atrP  entryTF       htfTF        sesStart sesEnd  corrGrp class
{"EURUSD",   10001,  15,     1.5,   3.0,   14,   PERIOD_M15,   PERIOD_H4,   8,       16,     1.0,    "forex"},
{"GBPUSD",   10002,  20,     1.5,   3.0,   14,   PERIOD_M15,   PERIOD_H4,   7,       16,     1.0,    "forex"},
{"USDJPY",   10003,  15,     1.5,   3.0,   14,   PERIOD_M15,   PERIOD_H4,   0,       9,      2.0,    "forex"},
{"AUDUSD",   10004,  20,     1.5,   3.0,   14,   PERIOD_M15,   PERIOD_H4,   22,      9,      3.0,    "forex"},
{"USDCAD",   10005,  20,     1.5,   3.0,   14,   PERIOD_M15,   PERIOD_H4,   12,      20,     4.0,    "forex"},
{"USDCHF",   10006,  20,     1.5,   3.0,   14,   PERIOD_M15,   PERIOD_H4,   7,       16,     1.0,    "forex"},
{"NZDUSD",   10007,  25,     1.5,   3.0,   14,   PERIOD_M15,   PERIOD_H4,   22,      9,      3.0,    "forex"},

// METALS
{"XAUUSD",   20001,  50,     1.5,   3.0,   14,   PERIOD_M15,   PERIOD_H4,   7,       9,      5.0,    "metal"},
{"XAGUSD",   20002,  100,    2.0,   3.5,   14,   PERIOD_M15,   PERIOD_H4,   7,       9,      5.0,    "metal"},

// INDICES
{"US500",    30001,  30,     1.5,   3.0,   14,   PERIOD_M15,   PERIOD_H4,   12,      20,     6.0,    "index"},
{"US100",    30002,  40,     1.5,   3.5,   14,   PERIOD_M15,   PERIOD_H4,   12,      20,     6.0,    "index"},
{"US30",     30003,  40,     1.5,   3.0,   14,   PERIOD_M15,   PERIOD_H4,   12,      20,     6.0,    "index"},
{"GER40",    30004,  30,     1.5,   3.0,   14,   PERIOD_M15,   PERIOD_H4,   7,       16,     7.0,    "index"},
{"UK100",    30005,  30,     1.5,   3.0,   14,   PERIOD_M15,   PERIOD_H4,   8,       16,     7.0,    "index"},

// CRYPTO CFDs (if broker offers)
{"BTCUSD",   40001,  200,    2.0,   4.0,   14,   PERIOD_H1,    PERIOD_D1,   0,       24,     8.0,    "crypto"},
{"ETHUSD",   40002,  150,    2.0,   4.0,   14,   PERIOD_H1,    PERIOD_D1,   0,       24,     8.0,    "crypto"},
};
```

---

## CMultiAssetManager Class

```mql5
class CMultiAssetManager {
private:
   SSymbolConfig  m_configs[];
   int            m_symbolCount;
   double         m_maxCorrelatedPositions;  // Max positions per correlation group
   double         m_totalPortfolioRiskPct;   // Max combined open risk %

public:
   CMultiAssetManager(double maxCorrPositions=2, double totalRiskPct=5.0) {
      m_maxCorrelatedPositions = maxCorrPositions;
      m_totalPortfolioRiskPct  = totalRiskPct;
      LoadDefaultProfiles();
   }

   //--- Get config for a specific symbol ---
   bool GetSymbolConfig(string symbol, SSymbolConfig &cfg) {
      for(int i = 0; i < m_symbolCount; i++) {
         if(m_configs[i].symbol == symbol) {
            cfg = m_configs[i];
            return true;
         }
      }
      return false;  // Symbol not configured
   }

   //--- Check if portfolio allows another position ---
   bool IsNewPositionAllowed(SSymbolConfig &newSymbolCfg) {
      // Rule 1: Max correlated positions
      int correlatedCount = CountPositionsInGroup(newSymbolCfg.correlationGroup);
      if(correlatedCount >= m_maxCorrelatedPositions) {
         PrintFormat("PORTFOLIO: Max correlated positions (%d) reached for group %.0f",
                     (int)m_maxCorrelatedPositions, newSymbolCfg.correlationGroup);
         return false;
      }

      // Rule 2: Total portfolio risk check
      double totalRisk = GetTotalOpenRiskPct();
      if(totalRisk >= m_totalPortfolioRiskPct) {
         PrintFormat("PORTFOLIO: Total risk %.1f%% at max %.1f%%",
                     totalRisk, m_totalPortfolioRiskPct);
         return false;
      }

      return true;
   }

   //--- Count open positions in correlation group ---
   int CountPositionsInGroup(double groupId) {
      int count = 0;
      for(int i = 0; i < PositionsTotal(); i++) {
         string sym = PositionGetSymbol(i);
         SSymbolConfig cfg;
         if(GetSymbolConfig(sym, cfg)) {
            if(cfg.correlationGroup == groupId) count++;
         }
      }
      return count;
   }

   //--- Calculate total open risk across all positions ---
   double GetTotalOpenRiskPct() {
      double totalRisk = 0;
      double balance   = AccountInfoDouble(ACCOUNT_BALANCE);
      for(int i = 0; i < PositionsTotal(); i++) {
         PositionGetString(POSITION_SYMBOL);
         double openPrice = PositionGetDouble(POSITION_PRICE_OPEN);
         double sl        = PositionGetDouble(POSITION_SL);
         double lots      = PositionGetDouble(POSITION_VOLUME);
         if(sl > 0) {
            double riskPts = MathAbs(openPrice - sl) / _Point;
            double tickVal = SymbolInfoDouble(PositionGetString(POSITION_SYMBOL),
                                              SYMBOL_TRADE_TICK_VALUE);
            double riskAmt = riskPts * tickVal * lots;
            totalRisk += (riskAmt / balance) * 100.0;
         }
      }
      return totalRisk;
   }

private:
   void LoadDefaultProfiles() {
      // Load from g_SymbolProfiles array above
      m_symbolCount = ArraySize(g_SymbolProfiles);
      ArrayResize(m_configs, m_symbolCount);
      ArrayCopy(m_configs, g_SymbolProfiles);
   }
};
```

---

## Multi-Symbol OnTick Architecture

```mql5
// In main EA file — iterate all configured symbols each tick
void OnTick() {
   if(!IsNewBar(PERIOD_M15)) return;

   for(int i = 0; i < ArraySize(g_SymbolProfiles); i++) {
      string sym = g_SymbolProfiles[i].symbol;

      // Check symbol is available on this broker
      if(!SymbolSelect(sym, true)) continue;

      // Run each symbol through the full 5-layer pipeline
      ProcessSymbol(sym, g_SymbolProfiles[i]);
   }
}

void ProcessSymbol(string sym, SSymbolConfig &cfg) {
   // Layer 0: Spread check
   double spread = (double)SymbolInfoInteger(sym, SYMBOL_SPREAD);
   if(spread > cfg.maxSpreadPoints) return;

   // Layer 1: Risk check (global)
   if(!g_RiskMgr.IsTradeAllowed()) return;

   // Layer 1b: Portfolio position check
   if(!g_PortfolioMgr.IsNewPositionAllowed(cfg)) return;

   // Layer 2: Regime filter (per symbol)
   ENUM_REGIME regime = g_Regime.GetRegimeForSymbol(sym, cfg.htfTF);
   if(regime == REGIME_DEAD || regime == REGIME_UNDEFINED) return;

   // Layer 3: Signal engine (per symbol)
   STradeSignal signal = g_Signal.GetSignalForSymbol(sym, cfg, regime);
   if(signal.type == SIGNAL_NONE) return;

   // Layer 1c: Lot size
   double lots = g_RiskMgr.CalculateLotSize(signal.slDistance);
   if(lots <= 0) return;

   // Layer 4: Execute
   g_TradeMgr.OpenTrade(sym, signal, lots, cfg.magicNumber);
   g_Analytics.LogTrade(sym, signal, lots);
}
```

---

## Correlation Groups (Reference)

```
Group 1.0 — USD Majors (EUR/GBP/CHF direction): EURUSD, GBPUSD, USDCHF
Group 2.0 — JPY pairs: USDJPY, EURJPY, GBPJPY
Group 3.0 — Commodity currencies: AUDUSD, NZDUSD
Group 4.0 — CAD: USDCAD
Group 5.0 — Metals: XAUUSD, XAGUSD
Group 6.0 — US Indices: US500, US100, US30 (max 1 position — highly correlated)
Group 7.0 — EU Indices: GER40, UK100
Group 8.0 — Crypto: BTCUSD, ETHUSD

Max 2 positions per group (except Group 6: max 1)
Max 5% total portfolio risk at any time
```

---

## Response Format for Multi-Asset Queries

1. Show the SSymbolConfig for the specific assets in question
2. Show correlation group assignment and position limits
3. Show total portfolio risk calculation
4. Provide the per-symbol ProcessSymbol() call pattern
5. Flag any broker-specific symbol naming issues (XAUUSD vs GOLD vs GOLDm)
