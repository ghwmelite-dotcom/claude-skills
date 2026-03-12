---
name: ea-trade-manager
description: >
  Advanced trade lifecycle management skill for MQL5 EAs. Use for ALL post-entry management:
  trailing stop loss (ATR-based, swing-based, chandelier, step), breakeven logic, partial
  close / scale-out strategies, take profit management (fixed, dynamic, multi-target),
  position scaling (pyramiding), trade modification with CTrade, open position management
  in OnTrade()/OnTick(), managing multiple open positions per symbol, and any "what happens
  after I enter a trade" logic. Trigger on: "trailing stop", "breakeven", "partial close",
  "scale out", "pyramiding", "take profit", "manage positions", "move SL", "CTrade",
  "position management", "trade modification", "multi-target TP". Every winning trade
  needs management — this skill extracts maximum profit from every position.
---

# EA Trade Manager Skill

Trade management is where edge is multiplied. Good entries with poor management
underperform. Average entries with excellent management outperform.

## Quick Reference Map

| Topic | Reference File |
|---|---|
| Trailing stop algorithms | `references/trailing-stops.md` |
| Breakeven & lock-in logic | `references/breakeven.md` |
| Partial close & scale-out | `references/partial-close.md` |
| Multi-target TP management | `references/multi-target-tp.md` |
| CTrade class full reference | `references/ctrade-reference.md` |

---

## CTradeManager Class (Core)

```mql5
#include <Trade\Trade.mqh>
#include <Trade\PositionInfo.mqh>

class CTradeManager : public IModule {
private:
   CTrade         m_trade;
   CPositionInfo  m_position;
   int            m_magicNumber;
   int            m_slippage;

   // Trade management parameters
   double         m_beActivationRR;   // RR to move SL to breakeven (e.g., 1.0)
   double         m_trailActivationRR;// RR to start trailing (e.g., 1.5)
   double         m_partialCloseRR;   // RR to take partial profit (e.g., 1.0)
   double         m_partialCloseSize; // % of position to close at TP1 (e.g., 0.5)
   bool           m_beApplied[];      // Track if BE applied per ticket

public:
   CTradeManager(int magic, double beRR=1.0, double trailRR=1.5,
                 double partialRR=1.0, double partialSize=0.5) {
      m_magicNumber      = magic;
      m_slippage         = 10;
      m_beActivationRR   = beRR;
      m_trailActivationRR= trailRR;
      m_partialCloseRR   = partialRR;
      m_partialCloseSize = partialSize;
      m_trade.SetExpertMagicNumber(magic);
      m_trade.SetDeviationInPoints(m_slippage);
      m_trade.SetTypeFilling(ORDER_FILLING_IOC);
   }

   //--- Open a trade from signal ---
   bool OpenTrade(STradeSignal &sig, double lots) {
      bool result = false;
      string comment = "Magic:" + IntegerToString(m_magicNumber) + " " + sig.reason;

      if(sig.type == SIGNAL_BUY) {
         result = m_trade.Buy(lots, _Symbol, 0, sig.stopLoss, sig.takeProfit, comment);
      } else if(sig.type == SIGNAL_SELL) {
         result = m_trade.Sell(lots, _Symbol, 0, sig.stopLoss, sig.takeProfit, comment);
      }

      if(!result) {
         PrintFormat("OpenTrade FAILED: Error %d — %s", GetLastError(),
                     m_trade.ResultRetcodeDescription());
      }
      return result;
   }

   //--- Called in OnTrade() to manage all open positions ---
   void ManageOpenTrades() {
      for(int i = PositionsTotal() - 1; i >= 0; i--) {
         if(!m_position.SelectByIndex(i)) continue;
         if(m_position.Magic() != m_magicNumber) continue;
         if(m_position.Symbol() != _Symbol) continue;

         double openPrice  = m_position.PriceOpen();
         double currentSL  = m_position.StopLoss();
         double currentTP  = m_position.TakeProfit();
         double currentPx  = m_position.PriceCurrent();
         double slDist     = MathAbs(openPrice - currentSL);
         double profit     = m_position.Profit();
         ulong  ticket     = m_position.Ticket();

         // --- Breakeven Logic ---
         ApplyBreakeven(ticket, openPrice, currentSL, currentTP,
                        currentPx, slDist);

         // --- ATR Trailing Stop ---
         ApplyATRTrail(ticket, openPrice, currentSL, currentTP,
                       currentPx, slDist);

         // --- Partial Close ---
         ApplyPartialClose(ticket, openPrice, currentSL, currentTP,
                           currentPx, slDist);
      }
   }

   //--- Breakeven: move SL to entry + spread when R:R reaches threshold ---
   void ApplyBreakeven(ulong ticket, double open, double sl, double tp,
                       double current, double slDist) {
      if(m_position.PositionType() == POSITION_TYPE_BUY) {
         double rr = (current - open) / slDist;
         if(rr >= m_beActivationRR && sl < open) {
            double newSL = open + SymbolInfoInteger(_Symbol, SYMBOL_SPREAD) *
                           SymbolInfoDouble(_Symbol, SYMBOL_POINT);
            m_trade.PositionModify(ticket, NormalizeDouble(newSL, _Digits), tp);
            PrintFormat("BE applied: ticket=%d SL moved to %.5f", ticket, newSL);
         }
      } else if(m_position.PositionType() == POSITION_TYPE_SELL) {
         double rr = (open - current) / slDist;
         if(rr >= m_beActivationRR && sl > open) {
            double newSL = open - SymbolInfoInteger(_Symbol, SYMBOL_SPREAD) *
                           SymbolInfoDouble(_Symbol, SYMBOL_POINT);
            m_trade.PositionModify(ticket, NormalizeDouble(newSL, _Digits), tp);
         }
      }
   }

   //--- ATR Trailing Stop ---
   void ApplyATRTrail(ulong ticket, double open, double sl, double tp,
                      double current, double slDist) {
      double atr    = iATR(_Symbol, PERIOD_H1, 14, 1);
      double trail  = atr * 1.5;  // Trail = 1.5x ATR

      if(m_position.PositionType() == POSITION_TYPE_BUY) {
         double rr    = (current - open) / slDist;
         double newSL = current - trail;
         if(rr >= m_trailActivationRR && newSL > sl) {
            m_trade.PositionModify(ticket, NormalizeDouble(newSL, _Digits), tp);
         }
      } else if(m_position.PositionType() == POSITION_TYPE_SELL) {
         double rr    = (open - current) / slDist;
         double newSL = current + trail;
         if(rr >= m_trailActivationRR && newSL < sl) {
            m_trade.PositionModify(ticket, NormalizeDouble(newSL, _Digits), tp);
         }
      }
   }

   //--- Partial Close: close 50% at 1:1 R:R ---
   void ApplyPartialClose(ulong ticket, double open, double sl, double tp,
                          double current, double slDist) {
      static ulong partialDoneTickets[];
      // Check if already partially closed
      for(int i = 0; i < ArraySize(partialDoneTickets); i++)
         if(partialDoneTickets[i] == ticket) return;

      double rr = 0;
      if(m_position.PositionType() == POSITION_TYPE_BUY)
         rr = (current - open) / slDist;
      else
         rr = (open - current) / slDist;

      if(rr >= m_partialCloseRR) {
         double lotsToClose = m_position.Volume() * m_partialCloseSize;
         double lotStep     = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_STEP);
         lotsToClose        = MathFloor(lotsToClose / lotStep) * lotStep;
         double minLot      = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_MIN);
         if(lotsToClose >= minLot) {
            m_trade.PositionClosePartial(ticket, lotsToClose);
            // Record ticket as done
            int sz = ArraySize(partialDoneTickets);
            ArrayResize(partialDoneTickets, sz + 1);
            partialDoneTickets[sz] = ticket;
            // Move SL to BE after partial close
            ApplyBreakeven(ticket, open, sl, tp, current, slDist);
         }
      }
   }

   //--- Close all positions (for daily halt, weekend, etc.) ---
   void CloseAllPositions(string reason="") {
      for(int i = PositionsTotal() - 1; i >= 0; i--) {
         if(m_position.SelectByIndex(i)) {
            if(m_position.Magic() == m_magicNumber) {
               m_trade.PositionClose(m_position.Ticket());
               Print("Closed position. Reason: ", reason);
            }
         }
      }
   }

   bool   IsReady()    { return true; }
   void   Reset()      { }
   string GetStatus()  { return "TradeManager: " + IntegerToString(PositionsTotal()) + " open"; }
};
```

---

## Trade Management Strategy Profiles

### Profile 1: Sniper (High RR, Low WR)
```
Partial Close: None — let full position run to 3:1 TP
Breakeven:     At 1.5:1 R:R
Trailing:      ATR trail starts at 2:1 R:R
Target:        3:1 R:R minimum
WR expectation: 40–50% acceptable at 3:1 RR
```

### Profile 2: Scalper (Quick profit, tight management)
```
Partial Close: 50% at 1:1, remainder at 2:1
Breakeven:     At 0.8:1 (very early — protect capital)
Trailing:      Tight 0.5x ATR trail
Target:        1.5:1 to 2:1 R:R
WR expectation: 60%+ needed at this RR
```

### Profile 3: Swing Trader (Multi-day positions)
```
Partial Close: 33% at 1:1, 33% at 2:1, 33% trail
Breakeven:     At 1:1
Trailing:      Wide 2x ATR trail — let winners run
Target:        3:1 to 5:1 R:R over days/weeks
WR expectation: 40% acceptable at 4:1 RR
```

---

## CTrade Error Handling

```mql5
bool SafeOrderSend(CTrade &trade, ENUM_TRADE_REQUEST_ACTIONS action) {
   int retries = 3;
   while(retries > 0) {
      bool result = (action == TRADE_ACTION_DEAL) ? /* execute */ true : false;
      if(result) return true;

      int error = GetLastError();
      switch(error) {
         case TRADE_RETCODE_REQUOTE:
         case TRADE_RETCODE_PRICE_CHANGED:
         case TRADE_RETCODE_OFF_QUOTES:
            Sleep(500);  // Brief pause, retry
            retries--;
            break;
         case TRADE_RETCODE_NO_MONEY:
         case TRADE_RETCODE_INVALID_VOLUME:
         case TRADE_RETCODE_INVALID_STOPS:
            return false;  // Non-recoverable — don't retry
         default:
            Print("Trade error: ", error, " — ", trade.ResultRetcodeDescription());
            return false;
      }
   }
   return false;
}
```

---

## Response Format for Trade Management Queries

1. Identify which management profile suits the strategy
2. Show the complete function with all edge cases handled
3. Flag the CTrade method used (PositionModify, PositionClose, PositionClosePartial)
4. Include minimum lot normalization in all partial close code
5. Always include error handling on every trade operation
