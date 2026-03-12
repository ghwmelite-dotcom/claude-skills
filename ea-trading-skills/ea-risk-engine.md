---
name: ea-risk-engine
description: >
  Indestructible risk management skill for MQL5 EAs. Use for ALL risk-related EA code:
  position sizing (fixed % risk, ATR-based, Kelly criterion), daily drawdown guards,
  total drawdown limits, equity curve protection, prop firm compliance (FTMO/MFF/E8),
  correlation-based exposure management, max open trades limits, lot size normalization,
  account balance tracking, equity stops, and any "how do I protect the account" question.
  Trigger on: "position size", "lot size", "drawdown", "risk", "prop firm", "FTMO",
  "daily loss limit", "max DD", "account protection", "equity stop", "risk management",
  "Kelly", "correlation risk", "exposure". This skill is the EA's immune system — never skip it.
---

# EA Risk Engine Skill

The risk engine is the most critical module. It is the last line of defense.
All trade execution is gated through this module. If this fails, everything fails.

## Quick Reference Map

| Topic | Reference File |
|---|---|
| Position sizing formulas | `references/position-sizing.md` |
| Drawdown tracking & halt logic | `references/drawdown-control.md` |
| Prop firm rule sets (FTMO, MFF, E8) | `references/prop-firm-rules.md` |
| Correlation & exposure management | `references/correlation-exposure.md` |
| Equity curve management | `references/equity-curve.md` |

---

## The CRiskManager Class (Complete Implementation)

```mql5
class CRiskManager : public IModule {
private:
   double   m_riskPercent;       // Risk per trade
   double   m_maxDailyDD;        // Max daily drawdown %
   double   m_maxTotalDD;        // Max total drawdown %
   double   m_dayStartBalance;   // Balance at session start
   double   m_peakBalance;       // Highest ever balance (for total DD)
   bool     m_haltTrading;       // Kill switch
   bool     m_reducedMode;       // Reduced lot mode (6% DD warning)
   int      m_consecutiveLosses; // Track losing streak
   datetime m_lastDayReset;      // Track daily reset

public:
   CRiskManager(double riskPct, double maxDailyDD, double maxTotalDD) {
      m_riskPercent        = riskPct;
      m_maxDailyDD         = maxDailyDD;
      m_maxTotalDD         = maxTotalDD;
      m_dayStartBalance    = AccountInfoDouble(ACCOUNT_BALANCE);
      m_peakBalance        = m_dayStartBalance;
      m_haltTrading        = false;
      m_reducedMode        = false;
      m_consecutiveLosses  = 0;
      m_lastDayReset       = TimeCurrent();
   }

   //--- Daily reset at midnight ---
   void DailyReset() {
      MqlDateTime now;
      TimeToStruct(TimeCurrent(), now);
      MqlDateTime last;
      TimeToStruct(m_lastDayReset, last);

      if(now.day != last.day) {
         m_dayStartBalance = AccountInfoDouble(ACCOUNT_BALANCE);
         m_haltTrading     = false;  // Unlock new trading day
         m_reducedMode     = false;
         m_lastDayReset    = TimeCurrent();
         Print("RiskManager: Daily reset. New balance: ", m_dayStartBalance);
      }
   }

   //--- Is trading allowed right now? ---
   bool IsTradeAllowed() {
      DailyReset();
      if(m_haltTrading) return false;

      double balance = AccountInfoDouble(ACCOUNT_BALANCE);
      double equity  = AccountInfoDouble(ACCOUNT_EQUITY);

      // Update peak
      if(balance > m_peakBalance) m_peakBalance = balance;

      // Daily drawdown check
      double dailyDD = (m_dayStartBalance - equity) / m_dayStartBalance * 100.0;
      if(dailyDD >= m_maxDailyDD) {
         m_haltTrading = true;
         Alert("RISK: Daily DD limit hit! EA halted for today. DD=", dailyDD, "%");
         return false;
      }

      // Total drawdown check
      double totalDD = (m_peakBalance - equity) / m_peakBalance * 100.0;
      if(totalDD >= m_maxTotalDD) {
         m_haltTrading = true;
         Alert("RISK: Total DD limit hit! EA permanently halted. DD=", totalDD, "%");
         return false;
      }

      // Warning zone — reduce lots
      m_reducedMode = (totalDD >= m_maxTotalDD * 0.7); // 70% of max DD
      return true;
   }

   //--- Calculate lot size ---
   double CalculateLotSize(double slDistancePoints) {
      if(slDistancePoints <= 0) return 0;

      double balance  = AccountInfoDouble(ACCOUNT_BALANCE);
      double riskPct  = m_reducedMode ? m_riskPercent * 0.5 : m_riskPercent;
      double riskAmt  = balance * riskPct / 100.0;

      double tickVal  = SymbolInfoDouble(_Symbol, SYMBOL_TRADE_TICK_VALUE);
      double tickSize = SymbolInfoDouble(_Symbol, SYMBOL_TRADE_TICK_SIZE);
      double lotStep  = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_STEP);
      double minLot   = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_MIN);
      double maxLot   = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_MAX);

      double lotSize = riskAmt / (slDistancePoints * tickVal / tickSize);
      lotSize = MathFloor(lotSize / lotStep) * lotStep;
      lotSize = MathMax(minLot, MathMin(maxLot, lotSize));

      return lotSize;
   }

   //--- Consecutive loss tracker ---
   void RecordTradeResult(bool isWin) {
      if(isWin) {
         m_consecutiveLosses = 0;
      } else {
         m_consecutiveLosses++;
         if(m_consecutiveLosses >= 3) {
            m_reducedMode = true;  // Enter reduced mode after 3 losses
            Print("RISK: 3 consecutive losses. Entering reduced mode.");
         }
      }
   }

   bool   IsReady()    { return true; }
   void   Reset()      { DailyReset(); }
   bool   IsHalted()   { return m_haltTrading; }
   bool   IsReduced()  { return m_reducedMode; }
   string GetStatus()  {
      return StringFormat("Risk: %.1f%% | DailyDD: %.2f%% | TotalDD: %.2f%% | Halt: %s",
         m_riskPercent,
         (m_dayStartBalance - AccountInfoDouble(ACCOUNT_EQUITY)) / m_dayStartBalance * 100,
         (m_peakBalance - AccountInfoDouble(ACCOUNT_EQUITY)) / m_peakBalance * 100,
         m_haltTrading ? "YES" : "NO");
   }
};
```

---

## Prop Firm Rule Sets

### FTMO Standard Challenge ($10K–$200K)
```
Daily Loss Limit:     5% of initial balance → halt at 4% (safety buffer)
Max Total Drawdown:   10% of initial balance → halt at 8% (safety buffer)
Min Trading Days:     4 days
Profit Target:        10% (Phase 1), 5% (Phase 2)
News Trading:         Allowed (check specific account)
Weekend Hold:         Allowed (check specific account)
EA Rule:              EAs allowed; no tick scalping abuse
```

### MyFundedFX (MFF)
```
Daily Loss Limit:     4% → EA halts at 3%
Max Total Drawdown:   8% → EA halts at 6.5%
Profit Target:        8% (aggressive), 6% (standard)
News Trading:         Check account type
```

### E8 Funding
```
Daily Loss Limit:     4% → EA halts at 3%
Max Total Drawdown:   8% → EA halts at 7%
Profit Target:        8%
Min Trading Days:     None
```

### The Funded Trader (TFT)
```
Daily Loss Limit:     5% → EA halts at 4%
Max Total Drawdown:   10% → EA halts at 8%
```

---

## Position Sizing Methods

### Method 1: Fixed Percentage Risk (Default)
```
Lots = (Balance × Risk%) / (SL_points × TickValue / TickSize)
```

### Method 2: ATR-Based Dynamic Risk
```mql5
double atr = iATR(_Symbol, PERIOD_H1, 14, 1);
double slPoints = atr * 1.5 / _Point;  // SL = 1.5x ATR
double lots = CalculateLotSize(slPoints);
// Benefit: wider ATR = automatically smaller position
```

### Method 3: Volatility-Scaled Risk
```mql5
// Scale risk down when volatility is elevated
double currentATR  = iATR(_Symbol, PERIOD_H1, 14, 1);
double historicATR = iATR(_Symbol, PERIOD_H1, 14, 20);  // 20-bar average
double volRatio    = currentATR / historicATR;
double adjustedRisk = m_riskPercent / volRatio;  // Less risk in high-vol
adjustedRisk = MathMax(0.25, MathMin(m_riskPercent, adjustedRisk));
```

### Method 4: Kelly Criterion (Advanced)
```
Kelly% = W - (1-W)/R
W = Win rate (decimal), R = Average Win / Average Loss

Example: 55% WR, 1.8 RR:
Kelly = 0.55 - (0.45/1.8) = 0.55 - 0.25 = 0.30 → 30% (use half-Kelly = 15%)
```

---

## Correlation-Based Exposure Management

```mql5
// Don't allow >2 highly correlated pairs simultaneously
// Correlation pairs to track:
// EUR/USD ↔ GBP/USD: +0.85 — count as same direction exposure
// AUD/USD ↔ NZD/USD: +0.88 — count as same direction exposure
// XAU/USD ↔ EUR/USD: +0.65 — partial correlation, allow both at half size
// US500  ↔ NAS100:   +0.95 — essentially same trade

bool IsCorrelatedPositionOpen(string symbol1, string symbol2) {
   for(int i = 0; i < PositionsTotal(); i++) {
      if(PositionGetSymbol(i) == symbol1 || PositionGetSymbol(i) == symbol2) {
         return true;
      }
   }
   return false;
}
```

---

## Response Format for Risk Queries

When answering any risk question:
1. State the lot size calculation with actual numbers
2. Show the DD guard that applies
3. Reference the applicable prop firm ruleset
4. Flag if current drawdown changes the allowed risk level
5. Always show both the formula AND the MQL5 implementation
