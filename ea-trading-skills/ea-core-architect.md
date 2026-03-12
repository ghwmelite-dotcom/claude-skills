---
name: ea-core-architect
description: >
  Master MQL5 Expert Advisor architecture skill. Use this skill for ALL EA structure,
  file organization, OOP design, MQL5 event model, CTrade usage, include file patterns,
  input parameter design, initialization/deinitialization logic, broker-agnostic coding,
  error handling patterns, and any question about HOW to build an EA from the ground up.
  Trigger on: "build an EA", "EA structure", "MQL5 architecture", "OnInit", "OnTick",
  "CTrade", "include files", "mqh", "Expert Advisor skeleton", "EA template", "OOP MQL5".
  Always use this skill before writing any EA code — architecture first, code second.
---

# EA Core Architect Skill

This skill governs the structural foundation of every EA built in this suite.
All other EA skills plug into this architecture. Read this first.

## The Golden Rule
**Architecture determines destiny.** A poorly structured EA cannot be made reliable
by clever strategy logic. Build the shell correctly once; all other modules slot in.

## Quick Reference Map

| Need | Reference File |
|---|---|
| File & folder structure, include patterns | `references/structure.md` |
| MQL5 OOP patterns, class design | `references/oop-patterns.md` |
| Event model, OnTick/OnTimer/OnTrade | `references/event-model.md` |
| Input parameters, configuration design | `references/inputs-config.md` |
| Error handling, logging, recovery | `references/error-handling.md` |

---

## Core Architecture Principles

### The 5-Layer EA Model
Every EA in this suite follows this layered model:

```
┌─────────────────────────────────┐
│  Layer 5: ANALYTICS & JOURNAL   │  → ea-analytics-hub skill
├─────────────────────────────────┤
│  Layer 4: TRADE MANAGEMENT      │  → ea-trade-manager skill
├─────────────────────────────────┤
│  Layer 3: SIGNAL ENGINE         │  → ea-signal-engine skill
├─────────────────────────────────┤
│  Layer 2: MARKET REGIME FILTER  │  → ea-market-regime skill
├─────────────────────────────────┤
│  Layer 1: RISK ENGINE           │  → ea-risk-engine skill
├─────────────────────────────────┤
│  Layer 0: CORE ARCHITECTURE     │  ← YOU ARE HERE (this skill)
└─────────────────────────────────┘
```

Each layer only communicates with adjacent layers. No layer skips levels.

### Mandatory File Structure
```
EA_Name/
├── EA_Name.mq5              # Main file — OnInit/OnTick/OnDeinit only
├── include/
│   ├── CoreEngine.mqh       # Base class all modules inherit from
│   ├── RiskManager.mqh      # Layer 1 — position sizing, DD control
│   ├── RegimeFilter.mqh     # Layer 2 — market state detection
│   ├── SignalEngine.mqh     # Layer 3 — entry/exit signal generation
│   ├── TradeManager.mqh     # Layer 4 — SL/TP/trailing management
│   ├── SessionFilter.mqh    # Session & news timing
│   ├── SymbolConfig.mqh     # Per-symbol parameters
│   ├── Analytics.mqh        # Layer 5 — logging, metrics
│   └── Utilities.mqh        # Shared helpers (time, math, formatting)
├── backtest/                # Backtest result archives
├── docs/
│   └── strategy_spec.md     # Strategy documentation
└── CLAUDE.md                # Claude Code project context
```

### Main EA File Template
```mql5
//+------------------------------------------------------------------+
//| EA_Name.mq5                                                        |
//| Copyright 2024, ohwpstudios                                        |
//| Architecture: 5-Layer Modular (ea-core-architect skill)           |
//+------------------------------------------------------------------+
#property copyright "ohwpstudios"
#property version   "1.00"
#property strict

// Include all modules
#include "include\RiskManager.mqh"
#include "include\RegimeFilter.mqh"
#include "include\SignalEngine.mqh"
#include "include\TradeManager.mqh"
#include "include\SessionFilter.mqh"
#include "include\Analytics.mqh"

// Input Parameters (defined in SymbolConfig.mqh defaults)
input group "=== RISK SETTINGS ==="
input double   InpRiskPercent    = 1.0;    // Risk per trade (%)
input double   InpMaxDailyDD     = 2.0;    // Max daily drawdown (%)
input double   InpMaxTotalDD     = 8.0;    // Max total drawdown (%)

input group "=== STRATEGY SETTINGS ==="
input int      InpATRPeriod      = 14;     // ATR period
input double   InpATRMultiplier  = 1.5;    // SL = ATR × multiplier
input double   InpRRRatio        = 2.0;    // Risk:Reward ratio

input group "=== SESSION SETTINGS ==="
input int      InpGMTOffset      = 0;      // Broker GMT offset
input bool     InpLondonSession  = true;   // Trade London session
input bool     InpNYSession      = true;   // Trade NY session

input group "=== EA SETTINGS ==="
input int      InpMagicNumber    = 10001;  // Magic number
input bool     InpEnableJournal  = true;   // Enable CSV journaling

// Module instances
CRiskManager    *g_RiskMgr;
CRegimeFilter   *g_Regime;
CSignalEngine   *g_Signal;
CTradeManager   *g_TradeMgr;
CSessionFilter  *g_Session;
CAnalytics      *g_Analytics;

//+------------------------------------------------------------------+
int OnInit() {
   // Instantiate all modules
   g_RiskMgr   = new CRiskManager(InpRiskPercent, InpMaxDailyDD, InpMaxTotalDD);
   g_Regime    = new CRegimeFilter();
   g_Signal    = new CSignalEngine(InpATRPeriod, InpATRMultiplier, InpRRRatio);
   g_TradeMgr  = new CTradeManager(InpMagicNumber);
   g_Session   = new CSessionFilter(InpGMTOffset, InpLondonSession, InpNYSession);
   g_Analytics = new CAnalytics(InpMagicNumber, InpEnableJournal);

   // Validate all modules initialized correctly
   if(!g_RiskMgr.IsReady() || !g_Signal.IsReady()) {
      Print("FATAL: Module initialization failed. EA stopping.");
      return INIT_FAILED;
   }

   g_Analytics.LogEvent("EA initialized on " + _Symbol);
   return INIT_SUCCEEDED;
}

//+------------------------------------------------------------------+
void OnTick() {
   // Gate 1: Only on new bar (prevents recalculation on every tick)
   if(!IsNewBar()) return;

   // Gate 2: Risk engine check (daily DD, total DD, halt state)
   if(!g_RiskMgr.IsTradeAllowed()) return;

   // Gate 3: Session filter
   if(!g_Session.IsActiveSession()) return;

   // Gate 4: Market regime filter
   ENUM_REGIME regime = g_Regime.GetCurrentRegime();
   if(regime == REGIME_UNDEFINED) return;

   // Gate 5: Generate signal
   STradeSignal signal = g_Signal.GetSignal(regime);
   if(signal.type == SIGNAL_NONE) return;

   // Gate 6: Calculate position size
   double lots = g_RiskMgr.CalculateLotSize(signal.slDistance);
   if(lots <= 0) return;

   // Execute trade
   if(g_TradeMgr.OpenTrade(signal, lots)) {
      g_Analytics.LogTrade(signal, lots);
   }
}

//+------------------------------------------------------------------+
void OnTrade() {
   g_TradeMgr.ManageOpenTrades();  // Trailing SL, BE, partial close
   g_Analytics.UpdateMetrics();
}

//+------------------------------------------------------------------+
void OnDeinit(const int reason) {
   g_Analytics.GenerateSummary();
   // Clean up memory
   delete g_RiskMgr;
   delete g_Regime;
   delete g_Signal;
   delete g_TradeMgr;
   delete g_Session;
   delete g_Analytics;
}
```

---

## Naming & Coding Conventions

```
Classes:      PascalCase prefix C  → CRiskManager, CSignalEngine
Interfaces:   PascalCase prefix I  → IModule, ISignalProvider
Enums:        UPPER_SNAKE          → ENUM_REGIME, SIGNAL_TYPE
Structs:      PascalCase prefix S  → STradeSignal, SBarData
Global vars:  g_ prefix            → g_RiskMgr, g_Signal
Input params: Inp prefix           → InpRiskPercent, InpMagicNumber
Local vars:   camelCase            → currentPrice, lotSize
Constants:    ALL_CAPS             → MAX_SLIPPAGE, MIN_LOT
Functions:    PascalCase           → CalculateLotSize(), IsNewBar()
```

---

## Base Module Interface (CoreEngine.mqh)

Every module MUST implement this interface:

```mql5
class IModule {
public:
   virtual bool   IsReady()   = 0;  // Returns false if init failed
   virtual void   Reset()     = 0;  // Resets module state (new day, new session)
   virtual string GetStatus() = 0;  // Returns human-readable status string
};
```

---

## Universal Helper: IsNewBar()

```mql5
bool IsNewBar(ENUM_TIMEFRAMES tf = PERIOD_CURRENT) {
   static datetime lastBarTime = 0;
   datetime currentBarTime = iTime(_Symbol, tf, 0);
   if(currentBarTime != lastBarTime) {
      lastBarTime = currentBarTime;
      return true;
   }
   return false;
}
```

Always gate OnTick() logic behind IsNewBar() unless you need tick-level precision (scalping).

---

## Broker-Agnostic Symbol Handling

```mql5
// In SymbolConfig.mqh — always use these, never hardcode
double g_Point    = SymbolInfoDouble(_Symbol, SYMBOL_POINT);
double g_LotStep  = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_STEP);
double g_MinLot   = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_MIN);
double g_MaxLot   = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_MAX);
int    g_Digits   = (int)SymbolInfoInteger(_Symbol, SYMBOL_DIGITS);
double g_TickVal  = SymbolInfoDouble(_Symbol, SYMBOL_TRADE_TICK_VALUE);
double g_TickSize = SymbolInfoDouble(_Symbol, SYMBOL_TRADE_TICK_SIZE);
double g_Spread   = (double)SymbolInfoInteger(_Symbol, SYMBOL_SPREAD) * g_Point;
```

Always normalize lot size before any order:
```mql5
double NormalizeLots(double lots) {
   lots = MathFloor(lots / g_LotStep) * g_LotStep;
   return MathMax(g_MinLot, MathMin(g_MaxLot, lots));
}
```

---

## Response Format for Architecture Queries

When asked to design or review EA structure:
1. Show the full file tree first
2. Show the main `.mq5` template with all gates
3. List each include module with its responsibility
4. Flag any architectural anti-patterns in existing code
5. Suggest refactoring path if legacy code exists
