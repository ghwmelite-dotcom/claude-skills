---
name: fintech-trading-systems
description: >
  Activates elite fintech and trading systems engineering capabilities. Use this skill for ANY
  task involving: trading dashboards and UIs; real-time market data (WebSocket feeds, OHLCV,
  order books, tick data); algorithmic trading logic and Expert Advisors (MQL5, Pine Script);
  portfolio tracking and P&L calculation; financial calculations (returns, Sharpe ratio,
  drawdown, risk metrics); candlestick charts and TradingView integration; broker API
  integration (MetaTrader, Alpaca, Interactive Brokers, Binance); order management systems;
  position sizing and risk management; backtesting frameworks; financial data pipelines; or any
  system where money, markets, or trading is involved. Trigger when the user mentions "trading",
  "market data", "EA", "Expert Advisor", "order book", "candlestick", "portfolio", "P&L",
  "drawdown", "backtesting", "broker", "Forex", "crypto", "stocks", or "financial data". In
  finance, correctness is non-negotiable — never approximate money calculations.
---

# Elite Fintech & Trading Systems Engineer

You build trading and financial systems where precision, reliability, and speed are not
optional. A rounding error is a financial error. Latency is money. Downtime is losses.

---

## PART 1 — FINANCIAL CALCULATIONS (CORRECTNESS FIRST)

### 1.1 Money Arithmetic — Never Use Floats

```typescript
// WRONG — floating point precision kills fintech
const price = 19.99;
const quantity = 3;
const total = price * quantity;  // 59.97000000000001 !!!

// RIGHT — always use integer cents or a Decimal library
import Decimal from 'decimal.js';

const priceDecimal = new Decimal('19.99');
const totalDecimal = priceDecimal.times(3);  // Decimal { value: '59.97' }

// For storage: store as integer cents
const priceCents = 1999;   // $19.99
const totalCents = priceCents * 3;   // 5997 cents = $59.97

// Formatting
const formatCurrency = (cents: number, currency = 'USD', locale = 'en-US') =>
  new Intl.NumberFormat(locale, {
    style: 'currency',
    currency,
    minimumFractionDigits: 2,
  }).format(cents / 100);
```

### 1.2 Core Trading Metrics

```typescript
// Return on Investment
const roi = (exitPrice: number, entryPrice: number): number =>
  ((exitPrice - entryPrice) / entryPrice) * 100;

// Compound Annual Growth Rate (CAGR)
const cagr = (startValue: number, endValue: number, years: number): number =>
  (Math.pow(endValue / startValue, 1 / years) - 1) * 100;

// Sharpe Ratio (risk-adjusted return)
const sharpeRatio = (returns: number[], riskFreeRate = 0.05): number => {
  const excessReturns = returns.map(r => r - riskFreeRate / 252);  // daily
  const mean = excessReturns.reduce((sum, r) => sum + r, 0) / excessReturns.length;
  const std = Math.sqrt(
    excessReturns.reduce((sum, r) => sum + Math.pow(r - mean, 2), 0) / excessReturns.length
  );
  return (mean / std) * Math.sqrt(252);  // Annualized
};

// Maximum Drawdown
const maxDrawdown = (equityCurve: number[]): number => {
  let peak = equityCurve[0];
  let maxDD = 0;
  for (const value of equityCurve) {
    if (value > peak) peak = value;
    const dd = (peak - value) / peak;
    if (dd > maxDD) maxDD = dd;
  }
  return maxDD * 100;  // As percentage
};

// Win Rate + Profit Factor
const tradingStats = (trades: { pnl: number }[]) => {
  const wins  = trades.filter(t => t.pnl > 0);
  const losses = trades.filter(t => t.pnl < 0);
  const grossProfit = wins.reduce((sum, t) => sum + t.pnl, 0);
  const grossLoss   = Math.abs(losses.reduce((sum, t) => sum + t.pnl, 0));

  return {
    winRate:      wins.length / trades.length,
    profitFactor: grossLoss > 0 ? grossProfit / grossLoss : Infinity,
    avgWin:       wins.length   > 0 ? grossProfit / wins.length   : 0,
    avgLoss:      losses.length > 0 ? grossLoss   / losses.length : 0,
    expectancy:   trades.reduce((sum, t) => sum + t.pnl, 0) / trades.length,
  };
};

// Position Sizing (Fixed Fractional Risk)
const positionSize = (
  accountBalance: number,
  riskPercent: number,     // 0.01 = 1%
  entryPrice: number,
  stopLoss: number
): number => {
  const riskAmount = accountBalance * riskPercent;
  const riskPerUnit = Math.abs(entryPrice - stopLoss);
  return Math.floor(riskAmount / riskPerUnit);
};
```

---

## PART 2 — REAL-TIME MARKET DATA

### 2.1 WebSocket Feed Handler (Durable Objects)

```typescript
// Durable Object for managing live market data subscriptions
export class MarketFeedDO implements DurableObject {
  private sessions: Map<string, WebSocket> = new Map();
  private latestData: Map<string, MarketData> = new Map();

  constructor(private state: DurableObjectState, private env: Env) {
    // Restore sessions on restart
    this.state.acceptWebSocket(/* ... */);
  }

  async fetch(request: Request): Promise<Response> {
    const url = new URL(request.url);

    if (request.headers.get('Upgrade') === 'websocket') {
      return this.handleWebSocket(request);
    }

    if (url.pathname === '/broadcast') {
      return this.handleBroadcast(request);
    }

    return new Response('Not found', { status: 404 });
  }

  private handleWebSocket(request: Request): Response {
    const [client, server] = Object.values(new WebSocketPair());

    const sessionId = crypto.randomUUID();
    this.state.acceptWebSocket(server, [sessionId]);

    // Send latest data on connect
    const snapshot = Object.fromEntries(this.latestData);
    server.send(JSON.stringify({ type: 'snapshot', data: snapshot }));

    return new Response(null, {
      status: 101,
      webSocket: client,
    });
  }

  async webSocketMessage(ws: WebSocket, message: string | ArrayBuffer): Promise<void> {
    const msg = JSON.parse(message as string);

    if (msg.type === 'subscribe') {
      const sessions = this.state.getWebSockets();
      ws.send(JSON.stringify({
        type: 'subscribed',
        symbols: msg.symbols,
        snapshot: msg.symbols.map((s: string) => this.latestData.get(s)).filter(Boolean),
      }));
    }
  }

  private async handleBroadcast(request: Request): Promise<Response> {
    const data: MarketData = await request.json();

    // Update latest
    this.latestData.set(data.symbol, data);

    // Fan out to all connected clients
    const sessions = this.state.getWebSockets();
    const message = JSON.stringify({ type: 'tick', data });

    for (const session of sessions) {
      try {
        session.send(message);
      } catch {
        // Client disconnected
      }
    }

    return new Response('OK');
  }

  async webSocketClose(ws: WebSocket): Promise<void> {
    // Clean up handled automatically by Durable Objects
  }
}
```

### 2.2 React Hook for Live Prices

```tsx
interface MarketDataState {
  prices: Record<string, number>;
  changes: Record<string, number>;
  isConnected: boolean;
  lastUpdated: Date | null;
}

export function useLiveMarketData(symbols: string[]) {
  const [state, setState] = useState<MarketDataState>({
    prices: {}, changes: {}, isConnected: false, lastUpdated: null,
  });
  const ws = useRef<WebSocket | null>(null);
  const reconnectTimer = useRef<ReturnType<typeof setTimeout>>();
  const reconnectDelay = useRef(1000);

  function connect() {
    ws.current = new WebSocket(`wss://api.myapp.com/market-feed`);

    ws.current.onopen = () => {
      setState(prev => ({ ...prev, isConnected: true }));
      reconnectDelay.current = 1000;  // Reset backoff on success

      ws.current!.send(JSON.stringify({ type: 'subscribe', symbols }));
    };

    ws.current.onmessage = (event) => {
      const msg = JSON.parse(event.data);

      if (msg.type === 'tick') {
        setState(prev => ({
          ...prev,
          prices: { ...prev.prices, [msg.data.symbol]: msg.data.price },
          changes: { ...prev.changes, [msg.data.symbol]: msg.data.changePercent },
          lastUpdated: new Date(),
        }));
      } else if (msg.type === 'snapshot') {
        const prices: Record<string, number> = {};
        const changes: Record<string, number> = {};
        for (const item of msg.data) {
          prices[item.symbol] = item.price;
          changes[item.symbol] = item.changePercent;
        }
        setState(prev => ({ ...prev, prices, changes, lastUpdated: new Date() }));
      }
    };

    ws.current.onclose = () => {
      setState(prev => ({ ...prev, isConnected: false }));
      // Exponential backoff reconnect
      reconnectTimer.current = setTimeout(() => {
        reconnectDelay.current = Math.min(reconnectDelay.current * 2, 30000);
        connect();
      }, reconnectDelay.current);
    };
  }

  useEffect(() => {
    connect();
    return () => {
      clearTimeout(reconnectTimer.current);
      ws.current?.close();
    };
  }, [symbols.join(',')]);

  return state;
}
```

---

## PART 3 — TRADING DASHBOARD UI

### 3.1 Price Ticker Component

```tsx
function PriceTicker({ symbol, price, change, prevPrice }: TickerProps) {
  const [flash, setFlash] = useState<'up' | 'down' | null>(null);

  // Flash effect on price change
  useEffect(() => {
    if (prevPrice === undefined || prevPrice === price) return;
    const direction = price > prevPrice ? 'up' : 'down';
    setFlash(direction);
    const timer = setTimeout(() => setFlash(null), 500);
    return () => clearTimeout(timer);
  }, [price, prevPrice]);

  const isPositive = change >= 0;

  return (
    <div className={cn(
      'flex items-center justify-between p-3 rounded-lg border transition-colors duration-500',
      flash === 'up'   && 'bg-green-500/10 border-green-500/30',
      flash === 'down' && 'bg-red-500/10 border-red-500/30',
      !flash           && 'bg-surface border-border',
    )}>
      <div>
        <span className="font-mono font-semibold text-sm">{symbol}</span>
      </div>
      <div className="text-right">
        <p className="font-mono font-semibold tabular-nums">
          {new Intl.NumberFormat('en-US', {
            style: 'currency',
            currency: 'USD',
            minimumFractionDigits: 2,
            maximumFractionDigits: 5,
          }).format(price)}
        </p>
        <p className={cn('text-xs font-mono tabular-nums', isPositive ? 'text-green-400' : 'text-red-400')}>
          {isPositive ? '+' : ''}{change.toFixed(2)}%
        </p>
      </div>
    </div>
  );
}
```

### 3.2 Order Book Component

```tsx
function OrderBook({ symbol }: { symbol: string }) {
  const { bids, asks } = useOrderBook(symbol);

  const maxSize = Math.max(
    ...bids.slice(0, 10).map(b => b.size),
    ...asks.slice(0, 10).map(a => a.size)
  );

  return (
    <div className="font-mono text-xs bg-surface rounded-xl border border-border overflow-hidden">
      <div className="px-4 py-2 border-b border-border">
        <span className="text-text-secondary">Order Book</span>
        <span className="ml-2 font-semibold">{symbol}</span>
      </div>

      {/* Headers */}
      <div className="grid grid-cols-3 px-4 py-1.5 text-text-secondary text-right border-b border-border">
        <span className="text-left">Price</span>
        <span>Size</span>
        <span>Total</span>
      </div>

      {/* Asks (sell orders) — shown in reverse (highest first) */}
      <div className="divide-y divide-border/50">
        {[...asks.slice(0, 10)].reverse().map((ask, i) => (
          <OrderRow key={ask.price} level={ask} side="ask" maxSize={maxSize} />
        ))}
      </div>

      {/* Spread */}
      <div className="px-4 py-2 bg-bg-subtle border-y border-border text-center">
        <span className="text-text-secondary">Spread: </span>
        <span className="tabular-nums">
          {(asks[0].price - bids[0].price).toFixed(5)}
        </span>
      </div>

      {/* Bids (buy orders) */}
      <div className="divide-y divide-border/50">
        {bids.slice(0, 10).map(bid => (
          <OrderRow key={bid.price} level={bid} side="bid" maxSize={maxSize} />
        ))}
      </div>
    </div>
  );
}

function OrderRow({ level, side, maxSize }: OrderRowProps) {
  const fillPct = (level.size / maxSize) * 100;
  const color = side === 'bid' ? 'rgb(34 197 94)' : 'rgb(239 68 68)';

  return (
    <div className="relative grid grid-cols-3 px-4 py-1 text-right tabular-nums">
      {/* Depth bar */}
      <div
        className="absolute inset-y-0 right-0"
        style={{
          width: `${fillPct}%`,
          background: `${color}15`,
        }}
      />
      <span className={cn('text-left z-10', side === 'bid' ? 'text-green-400' : 'text-red-400')}>
        {level.price.toFixed(5)}
      </span>
      <span className="z-10">{level.size.toFixed(2)}</span>
      <span className="z-10 text-text-secondary">{level.total.toFixed(2)}</span>
    </div>
  );
}
```

---

## PART 4 — EXPERT ADVISOR (MQL5)

### 4.1 EA Structure Template

```mql5
//+------------------------------------------------------------------+
//| EA: MyStrategy.mq5                                               |
//| Copyright 2025, YourName                                         |
//+------------------------------------------------------------------+
#property copyright "2025"
#property version   "1.00"
#property strict

#include <Trade\Trade.mqh>
#include <Trade\PositionInfo.mqh>

// — Input Parameters —
input group "Risk Management"
input double InpRiskPercent   = 1.0;    // Risk per trade (%)
input double InpMaxDailyLoss  = 5.0;    // Max daily loss (%)
input int    InpMaxPositions  = 3;      // Max simultaneous positions

input group "Strategy Parameters"
input int    InpFastPeriod    = 10;     // Fast MA period
input int    InpSlowPeriod    = 21;     // Slow MA period
input ENUM_TIMEFRAMES InpTF   = PERIOD_H1;  // Timeframe

input group "Execution"
input double InpSlippage      = 3;     // Max slippage (points)
input bool   InpUseTrailing   = true;  // Use trailing stop

// — Global Variables —
CTrade      trade;
int         fastMAHandle, slowMAHandle;
double      fastMABuffer[], slowMABuffer[];
datetime    lastBarTime;
double      dailyOpenBalance;

//+------------------------------------------------------------------+
//| Expert initialization                                             |
//+------------------------------------------------------------------+
int OnInit() {
    trade.SetExpertMagicNumber(123456);
    trade.SetDeviationInPoints((ulong)InpSlippage);
    trade.SetTypeFilling(ORDER_FILLING_FOK);

    fastMAHandle = iMA(_Symbol, InpTF, InpFastPeriod, 0, MODE_EMA, PRICE_CLOSE);
    slowMAHandle = iMA(_Symbol, InpTF, InpSlowPeriod, 0, MODE_EMA, PRICE_CLOSE);

    if (fastMAHandle == INVALID_HANDLE || slowMAHandle == INVALID_HANDLE) {
        Print("ERROR: Failed to create MA indicators");
        return INIT_FAILED;
    }

    ArraySetAsSeries(fastMABuffer, true);
    ArraySetAsSeries(slowMABuffer, true);

    dailyOpenBalance = AccountInfoDouble(ACCOUNT_BALANCE);
    Print("EA initialized. Balance: ", dailyOpenBalance);
    return INIT_SUCCEEDED;
}

//+------------------------------------------------------------------+
//| Expert tick function                                              |
//+------------------------------------------------------------------+
void OnTick() {
    // Only run on new bar
    datetime currentBarTime = iTime(_Symbol, InpTF, 0);
    if (currentBarTime == lastBarTime) return;
    lastBarTime = currentBarTime;

    // Daily loss check
    double currentBalance = AccountInfoDouble(ACCOUNT_BALANCE);
    double dailyLoss = (dailyOpenBalance - currentBalance) / dailyOpenBalance * 100;
    if (dailyLoss >= InpMaxDailyLoss) {
        Print("Daily loss limit reached: ", DoubleToString(dailyLoss, 2), "%");
        return;
    }

    // Get indicator values
    if (CopyBuffer(fastMAHandle, 0, 0, 3, fastMABuffer) < 3) return;
    if (CopyBuffer(slowMAHandle, 0, 0, 3, slowMABuffer) < 3) return;

    // Signal detection
    bool bullishCross = fastMABuffer[1] <= slowMABuffer[1] && fastMABuffer[0] > slowMABuffer[0];
    bool bearishCross = fastMABuffer[1] >= slowMABuffer[1] && fastMABuffer[0] < slowMABuffer[0];

    int openPositions = CountOwnPositions();

    if (bullishCross && openPositions < InpMaxPositions) {
        OpenLong();
    } else if (bearishCross && openPositions < InpMaxPositions) {
        OpenShort();
    }

    // Manage open positions
    if (InpUseTrailing) ManageTrailingStops();
}

//+------------------------------------------------------------------+
int CountOwnPositions() {
    int count = 0;
    for (int i = PositionsTotal() - 1; i >= 0; i--) {
        if (PositionGetTicket(i) > 0 &&
            PositionGetString(POSITION_SYMBOL) == _Symbol &&
            PositionGetInteger(POSITION_MAGIC) == 123456) {
            count++;
        }
    }
    return count;
}

double CalcLotSize(double stopDistancePoints) {
    double riskAmount    = AccountInfoDouble(ACCOUNT_BALANCE) * InpRiskPercent / 100.0;
    double tickValue     = SymbolInfoDouble(_Symbol, SYMBOL_TRADE_TICK_VALUE);
    double tickSize      = SymbolInfoDouble(_Symbol, SYMBOL_TRADE_TICK_SIZE);
    double lotStep       = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_STEP);
    double minLot        = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_MIN);
    double maxLot        = SymbolInfoDouble(_Symbol, SYMBOL_VOLUME_MAX);

    if (tickValue == 0 || tickSize == 0 || stopDistancePoints == 0) return minLot;

    double lot = riskAmount / (stopDistancePoints * tickValue / tickSize);
    lot = MathFloor(lot / lotStep) * lotStep;

    return MathMax(minLot, MathMin(maxLot, lot));
}
```

---

## PART 5 — FINANCIAL DATA PIPELINE

### 5.1 Market Data Ingestion Worker

```typescript
// Cron worker: fetch and store OHLCV data
export default {
  async scheduled(event: ScheduledEvent, env: Env, ctx: ExecutionContext): Promise<void> {
    ctx.waitUntil(ingestMarketData(env));
  }
};

async function ingestMarketData(env: Env): Promise<void> {
  const symbols = await getWatchlistSymbols(env);

  const results = await Promise.allSettled(
    symbols.map(symbol => fetchAndStoreOHLCV(symbol, '1h', env))
  );

  const failures = results.filter(r => r.status === 'rejected');
  if (failures.length > 0) {
    console.error(`Failed to ingest ${failures.length}/${symbols.length} symbols`);
  }
}

async function fetchAndStoreOHLCV(symbol: string, interval: string, env: Env): Promise<void> {
  const data = await fetchFromDataProvider(symbol, interval, env);

  const statements = data.candles.map(candle =>
    env.DB.prepare(`
      INSERT OR REPLACE INTO ohlcv (symbol, interval, time, open, high, low, close, volume)
      VALUES (?, ?, ?, ?, ?, ?, ?, ?)
    `).bind(symbol, interval, candle.time, candle.open, candle.high, candle.low, candle.close, candle.volume)
  );

  await env.DB.batch(statements);
}
```

---

## PART 6 — TRADING SYSTEM CHECKLIST

```
Data integrity:
□ All money stored as integers (cents, satoshis, base units)
□ All calculations use Decimal library (no native float arithmetic)
□ Numbers formatted with Intl.NumberFormat (locale-aware)
□ Timestamps stored as Unix seconds (not milliseconds) for consistency

Real-time:
□ WebSocket reconnect with exponential backoff
□ Heartbeat ping every 30s to detect stale connections
□ Sequence numbers for detecting dropped messages
□ Snapshot on reconnect (don't rely on incremental updates after gap)

Risk management:
□ Position sizing logic server-side (not just UI)
□ Max daily loss limit enforced
□ Max position count enforced
□ Stop-loss required for all trades
□ Margin call detection and notification

UI:
□ Price flash animation on tick update (green up, red down)
□ All numbers monospaced font with tabular-nums
□ Spreads and P&L formatted to correct decimal places per asset
□ Loading states for all async data (skeleton, not spinners)
□ Offline / disconnected state clearly communicated
```

---

> **Core Principle**: In trading systems, a bug is a financial loss.
> There are no acceptable rounding errors, no acceptable race conditions,
> no acceptable downtime during market hours. Test like money is on the line — because it is.
