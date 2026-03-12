---
name: ea-analytics-hub
description: >
  Complete trade analytics, journaling, and performance monitoring skill for MQL5 EAs.
  Use for ALL logging, reporting, and performance tracking code: CSV trade journal generation,
  equity curve tracking, real-time dashboard (OnChartEvent), performance metrics calculation
  inside EA (WR, PF, Sharpe, DD tracking), trade summary reports, daily/weekly P&L summaries,
  alert systems, magic number filtering for analytics, and any "how do I track/report/measure
  EA performance in real-time" question. Trigger on: "journal", "logging", "CSV", "dashboard",
  "performance tracking", "equity curve", "analytics", "report", "OnChartEvent", "metrics",
  "daily summary", "P&L", "trade log", "DrawDown tracking", "real-time monitor". What gets
  measured gets improved — this skill makes the EA self-aware.
---

# EA Analytics Hub Skill

An EA without analytics is flying blind. This skill makes the EA fully self-aware —
logging every trade, tracking every metric, and surfacing problems before they
become account-destroying drawdowns.

## Quick Reference Map

| Topic | Reference File |
|---|---|
| CSV journal implementation | `references/csv-journal.md` |
| Real-time dashboard (chart) | `references/chart-dashboard.md` |
| Performance metrics calculation | `references/perf-metrics.md` |
| Alert & notification system | `references/alerts.md` |
| Equity curve visualization | `references/equity-curve.md` |

---

## CAnalytics Class (Complete)

```mql5
class CAnalytics : public IModule {
private:
   int      m_magicNumber;
   bool     m_enableJournal;
   string   m_journalPath;
   int      m_fileHandle;

   // Running metrics
   int      m_totalTrades;
   int      m_winCount;
   int      m_lossCount;
   double   m_grossProfit;
   double   m_grossLoss;
   double   m_peakEquity;
   double   m_maxDrawdown;
   double   m_totalPips;
   double   m_startBalance;
   datetime m_startTime;

   // Per-session tracking
   int      m_dailyTrades;
   double   m_dailyPnL;

public:
   CAnalytics(int magic, bool enableJournal=true) {
      m_magicNumber   = magic;
      m_enableJournal = enableJournal;
      m_startBalance  = AccountInfoDouble(ACCOUNT_BALANCE);
      m_peakEquity    = m_startBalance;
      m_startTime     = TimeCurrent();
      m_totalTrades   = 0;
      m_winCount      = 0;
      m_lossCount     = 0;
      m_grossProfit   = 0;
      m_grossLoss     = 0;
      m_maxDrawdown   = 0;
      m_totalPips     = 0;
      m_dailyTrades   = 0;
      m_dailyPnL      = 0;

      if(m_enableJournal) InitJournal();
   }

   //--- Initialize CSV journal ---
   void InitJournal() {
      m_journalPath = "EA_Journal_" + IntegerToString(m_magicNumber) +
                      "_" + TimeToString(TimeCurrent(), TIME_DATE) + ".csv";
      m_fileHandle = FileOpen(m_journalPath, FILE_WRITE|FILE_CSV|FILE_ANSI, ',');
      if(m_fileHandle == INVALID_HANDLE) {
         Print("ANALYTICS: Failed to open journal file!");
         m_enableJournal = false;
         return;
      }
      // Write CSV header
      FileWrite(m_fileHandle,
         "Ticket", "Symbol", "Direction", "OpenTime", "CloseTime",
         "EntryPrice", "ExitPrice", "StopLoss", "TakeProfit",
         "Lots", "Pips", "PnL_USD", "RR_Actual", "Signal_Reason",
         "Regime", "Session", "DailyTradeCount", "DrawdownPct"
      );
   }

   //--- Log a new trade entry ---
   void LogTrade(STradeSignal &sig, double lots) {
      if(!m_enableJournal || m_fileHandle == INVALID_HANDLE) return;

      m_totalTrades++;
      m_dailyTrades++;
      double currentDD = GetCurrentDrawdownPct();

      FileWrite(m_fileHandle,
         "OPEN",
         _Symbol,
         sig.type == SIGNAL_BUY ? "BUY" : "SELL",
         TimeToString(TimeCurrent(), TIME_DATE|TIME_MINUTES),
         "",  // CloseTime — filled on close
         DoubleToString(sig.entryPrice, _Digits),
         "",  // ExitPrice
         DoubleToString(sig.stopLoss, _Digits),
         DoubleToString(sig.takeProfit, _Digits),
         DoubleToString(lots, 2),
         "",  // Pips
         "",  // PnL
         "",  // RR actual
         sig.reason,
         "",  // Regime
         "",  // Session
         IntegerToString(m_dailyTrades),
         DoubleToString(currentDD, 2)
      );
   }

   //--- Log trade close (call from OnTradeTransaction) ---
   void LogTradeClose(ulong ticket) {
      if(!m_enableJournal || m_fileHandle == INVALID_HANDLE) return;
      if(!HistoryDealSelect(ticket)) return;

      double profit  = HistoryDealGetDouble(ticket, DEAL_PROFIT) +
                       HistoryDealGetDouble(ticket, DEAL_SWAP) +
                       HistoryDealGetDouble(ticket, DEAL_COMMISSION);
      double pips    = CalculatePipsFromDeal(ticket);
      bool   isWin   = (profit > 0);

      // Update running metrics
      if(isWin) { m_winCount++; m_grossProfit += profit; }
      else      { m_lossCount++; m_grossLoss  += MathAbs(profit); }
      m_totalPips += pips;
      m_dailyPnL  += profit;
      UpdateDrawdown();

      FileWrite(m_fileHandle,
         "CLOSE",
         HistoryDealGetString(ticket, DEAL_SYMBOL),
         HistoryDealGetInteger(ticket, DEAL_TYPE) == DEAL_TYPE_BUY ? "BUY" : "SELL",
         "",
         TimeToString((datetime)HistoryDealGetInteger(ticket, DEAL_TIME),
                      TIME_DATE|TIME_MINUTES),
         "",
         DoubleToString(HistoryDealGetDouble(ticket, DEAL_PRICE), _Digits),
         "", "",
         DoubleToString(HistoryDealGetDouble(ticket, DEAL_VOLUME), 2),
         DoubleToString(pips, 1),
         DoubleToString(profit, 2),
         "",
         "", "", "",
         DoubleToString(GetCurrentDrawdownPct(), 2)
      );
   }

   //--- Update metrics on every tick ---
   void UpdateMetrics() {
      double equity = AccountInfoDouble(ACCOUNT_EQUITY);
      if(equity > m_peakEquity) m_peakEquity = equity;
      double dd = (m_peakEquity - equity) / m_peakEquity * 100.0;
      if(dd > m_maxDrawdown) m_maxDrawdown = dd;
   }

   //--- Generate end-of-session summary ---
   void GenerateSummary() {
      double pf  = m_grossLoss > 0 ? m_grossProfit / m_grossLoss : 0;
      double wr  = m_totalTrades > 0 ? (double)m_winCount / m_totalTrades * 100.0 : 0;
      double net = m_grossProfit - m_grossLoss;

      string summary = StringFormat(
         "\n========== EA PERFORMANCE SUMMARY ==========\n"
         "Period:         %s to %s\n"
         "Total Trades:   %d (W: %d | L: %d)\n"
         "Win Rate:       %.1f%%\n"
         "Profit Factor:  %.2f\n"
         "Net Profit:     $%.2f\n"
         "Total Pips:     %.1f\n"
         "Max Drawdown:   %.2f%%\n"
         "Recovery Factor:%.2f\n"
         "============================================",
         TimeToString(m_startTime, TIME_DATE),
         TimeToString(TimeCurrent(), TIME_DATE),
         m_totalTrades, m_winCount, m_lossCount,
         wr, pf, net, m_totalPips, m_maxDrawdown,
         net > 0 ? net / m_maxDrawdown : 0
      );
      Print(summary);
      SendNotification(summary);  // Push notification to MT5 mobile

      if(m_fileHandle != INVALID_HANDLE) {
         FileWrite(m_fileHandle, summary);
         FileClose(m_fileHandle);
      }
   }

   //--- Log events (init, errors, regime changes) ---
   void LogEvent(string message) {
      string logLine = TimeToString(TimeCurrent(), TIME_DATE|TIME_MINUTES) +
                       " | " + message;
      Print(logLine);
      if(m_enableJournal && m_fileHandle != INVALID_HANDLE)
         FileWrite(m_fileHandle, "EVENT", logLine);
   }

private:
   double GetCurrentDrawdownPct() {
      double equity = AccountInfoDouble(ACCOUNT_EQUITY);
      return (m_peakEquity - equity) / m_peakEquity * 100.0;
   }

   void UpdateDrawdown() {
      double dd = GetCurrentDrawdownPct();
      if(dd > m_maxDrawdown) m_maxDrawdown = dd;
   }

   double CalculatePipsFromDeal(ulong ticket) {
      // Simplified — expand with symbol-specific pip calculation
      return 0;
   }

public:
   bool   IsReady()    { return true; }
   void   Reset()      { m_dailyTrades = 0; m_dailyPnL = 0; }
   string GetStatus()  {
      return StringFormat("Analytics: Trades=%d WR=%.0f%% PF=%.2f DD=%.1f%%",
         m_totalTrades,
         m_totalTrades > 0 ? (double)m_winCount/m_totalTrades*100 : 0,
         m_grossLoss > 0 ? m_grossProfit / m_grossLoss : 0,
         m_maxDrawdown);
   }
};
```

---

## Real-Time Chart Dashboard

```mql5
// Display live metrics on chart as a comment (simple)
void UpdateChartDisplay() {
   double equity   = AccountInfoDouble(ACCOUNT_EQUITY);
   double balance  = AccountInfoDouble(ACCOUNT_BALANCE);
   double floatPnL = equity - balance;

   string dashboard = StringFormat(
      "═══════════════════════════════\n"
      "  EA: Elite_v1  |  Magic: %d\n"
      "═══════════════════════════════\n"
      "  Balance:   $%.2f\n"
      "  Equity:    $%.2f (%+.2f)\n"
      "  Open DD:   %.2f%%\n"
      "  Max DD:    %.2f%%\n"
      "─────────────────────────────\n"
      "  Today:     %d trades | $%+.2f\n"
      "  Total:     %d trades\n"
      "  Win Rate:  %.1f%%\n"
      "  PF:        %.2f\n"
      "─────────────────────────────\n"
      "  Regime:    %s\n"
      "  Session:   %s\n"
      "  Status:    %s\n"
      "═══════════════════════════════",
      m_magicNumber,
      balance, equity, floatPnL,
      GetCurrentDrawdownPct(), m_maxDrawdown,
      m_dailyTrades, m_dailyPnL,
      m_totalTrades,
      m_totalTrades > 0 ? (double)m_winCount/m_totalTrades*100 : 0,
      m_grossLoss > 0 ? m_grossProfit / m_grossLoss : 0,
      "TRENDING_BULL",   // Pass from regime filter
      "London Open",     // Pass from session filter
      "ACTIVE"           // or HALTED
   );
   Comment(dashboard);
}
```

---

## Alert & Notification System

```mql5
void SendTradeAlert(string message, bool pushNotification=true) {
   string fullMsg = "[EA:" + IntegerToString(m_magicNumber) + "] " + message;
   Print(fullMsg);
   if(pushNotification) SendNotification(fullMsg);  // MT5 mobile push
}

// Key alert triggers:
// 1. Trade opened/closed
// 2. Daily DD warning (at 1.5% of 2% limit)
// 3. Total DD warning (at 6% of 8% limit)
// 4. EA halted (DD limit breached)
// 5. 3 consecutive losses (reduced mode activated)
// 6. New ATH equity reached
```

---

## Metrics Dashboard — What to Always Track

```
REAL-TIME (updated every tick):
  - Current equity & floating P&L
  - Open drawdown %
  - Today's trade count and P&L
  - Current regime and session
  - EA status (active/halted/reduced)

DAILY SUMMARY (on session close):
  - Total trades, WR, PF
  - Day's P&L in $ and pips
  - Running max drawdown
  - Consecutive win/loss streak

CUMULATIVE (full session):
  - Net profit
  - Max drawdown
  - Recovery factor
  - Total pips
  - Sharpe ratio estimate
```

---

## Response Format for Analytics Queries

1. Identify which metric or logging feature is needed
2. Show the complete CAnalytics method, not just a snippet
3. Include file handle null checks on all file operations
4. Show how to call the method from OnTick / OnTrade / OnDeinit
5. Always include the chart display update alongside any metric addition
