---
name: data-visualization
description: >
  Activates elite data visualization and analytics dashboard engineering. Use this skill for
  ANY task involving: building charts, graphs, or dashboards (Recharts, D3.js, Chart.js,
  Plotly, Vega-Lite); real-time data displays; financial charts (TradingView Lightweight Charts,
  candlestick, OHLCV); map visualizations (Mapbox GL JS, Leaflet, Deck.gl); heatmaps; time-series
  data; KPI dashboards; analytics reporting UIs; data tables with sorting/filtering; sparklines;
  progress indicators; gauge charts; or any UI where numbers and data need to be communicated
  visually. Trigger when the user mentions "chart", "graph", "dashboard", "analytics", "metrics",
  "visualization", "map", "heatmap", "real-time data", "KPI", or "reports". Always prioritize
  clarity over decoration — the best chart is the one that answers the question instantly.
---

# Elite Data Visualization Engineer

You build data visualizations that communicate truth clearly and beautifully. You understand
chart selection, color theory for data, accessibility in visualization, and performance with
large datasets.

---

## PART 1 — CHART SELECTION FRAMEWORK

### 1.1 Choosing the Right Chart

```
Comparison (how things differ):
  2-8 categories, exact values matter → Bar chart (vertical)
  Many categories, rank matters       → Horizontal bar chart
  Change over time (few series)       → Line chart
  Change over time (many series)      → Area chart (stacked)
  Part-to-whole with few parts        → Donut chart (max 5 segments)
  Part-to-whole, exact values matter  → Stacked bar chart

Distribution (how data is spread):
  Single variable, many values        → Histogram
  Comparing two distributions         → Box plot
  Density of points                   → Scatter plot / bubble chart

Relationship (correlation):
  Two variables                       → Scatter plot
  Three variables                     → Bubble chart
  Many variables                      → Heatmap / correlation matrix

Financial:
  Price over time                     → OHLCV candlestick
  Price + volume                      → Candlestick + volume bars
  Portfolio performance               → Line chart with baseline

Geospatial:
  Data by region                      → Choropleth map
  Point data                          → Marker cluster map
  Flow/movement                       → Arc map
  Density                             → Heatmap overlay

AVOID:
  × 3D charts — distort perception
  × Pie charts with > 5 segments — impossible to compare
  × Dual-axis charts — mislead by scaling
  × Truncated Y-axis — exaggerates differences
```

---

## PART 2 — RECHARTS (REACT)

### 2.1 Production-Ready Line Chart

```tsx
import {
  LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip,
  Legend, ResponsiveContainer, ReferenceLine,
} from 'recharts';
import { format } from 'date-fns';

interface DataPoint {
  timestamp: number;
  revenue: number;
  target: number;
}

function RevenueChart({ data }: { data: DataPoint[] }) {
  const formatCurrency = (value: number) =>
    new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD',
      notation: 'compact', maximumFractionDigits: 1 }).format(value);

  const formatDate = (timestamp: number) => format(new Date(timestamp), 'MMM d');

  return (
    <div className="w-full">
      <h3 className="text-sm font-medium text-text-secondary mb-4">Revenue vs Target</h3>
      <ResponsiveContainer width="100%" height={300}>
        <LineChart data={data} margin={{ top: 5, right: 20, bottom: 5, left: 0 }}>
          <CartesianGrid
            strokeDasharray="3 3"
            stroke="var(--color-border)"
            vertical={false}
          />
          <XAxis
            dataKey="timestamp"
            tickFormatter={formatDate}
            axisLine={false}
            tickLine={false}
            tick={{ fontSize: 12, fill: 'var(--color-text-secondary)' }}
            dy={8}
          />
          <YAxis
            tickFormatter={formatCurrency}
            axisLine={false}
            tickLine={false}
            tick={{ fontSize: 12, fill: 'var(--color-text-secondary)' }}
            width={64}
          />
          <Tooltip
            content={<CustomTooltip formatter={formatCurrency} />}
          />
          <Legend
            content={<CustomLegend />}
            verticalAlign="top"
            align="right"
          />
          <Line
            type="monotone"
            dataKey="revenue"
            stroke="var(--color-brand)"
            strokeWidth={2}
            dot={false}
            activeDot={{ r: 4, strokeWidth: 0 }}
            name="Revenue"
          />
          <Line
            type="monotone"
            dataKey="target"
            stroke="var(--color-text-disabled)"
            strokeWidth={1.5}
            strokeDasharray="4 4"
            dot={false}
            name="Target"
          />
        </LineChart>
      </ResponsiveContainer>
    </div>
  );
}

// Custom tooltip with proper styling
function CustomTooltip({ active, payload, label, formatter }: any) {
  if (!active || !payload?.length) return null;

  return (
    <div className="bg-surface border border-border rounded-lg shadow-lg p-3 text-sm">
      <p className="text-text-secondary mb-2">{format(new Date(label), 'MMM d, yyyy')}</p>
      {payload.map((entry: any) => (
        <div key={entry.dataKey} className="flex items-center gap-2">
          <div className="w-2 h-2 rounded-full" style={{ backgroundColor: entry.color }} />
          <span className="text-text-secondary">{entry.name}:</span>
          <span className="font-medium text-text-primary">{formatter(entry.value)}</span>
        </div>
      ))}
    </div>
  );
}
```

### 2.2 KPI Dashboard Components

```tsx
interface KPICardProps {
  title: string;
  value: string | number;
  change: number;       // percentage change
  changeLabel?: string;
  sparklineData?: number[];
  prefix?: string;
  suffix?: string;
}

function KPICard({ title, value, change, changeLabel, sparklineData, prefix, suffix }: KPICardProps) {
  const isPositive = change >= 0;

  return (
    <div className="bg-surface border border-border rounded-xl p-5">
      <div className="flex items-start justify-between mb-3">
        <p className="text-sm text-text-secondary">{title}</p>
        <span className={cn(
          'inline-flex items-center gap-1 text-xs font-medium px-1.5 py-0.5 rounded-full',
          isPositive
            ? 'bg-green-100 text-green-700 dark:bg-green-900/30 dark:text-green-400'
            : 'bg-red-100 text-red-700 dark:bg-red-900/30 dark:text-red-400'
        )}>
          {isPositive ? <TrendingUp className="h-3 w-3" /> : <TrendingDown className="h-3 w-3" />}
          {Math.abs(change).toFixed(1)}%
        </span>
      </div>

      <p className="text-2xl font-semibold text-text-primary">
        {prefix}{typeof value === 'number'
          ? new Intl.NumberFormat().format(value)
          : value}{suffix}
      </p>

      {changeLabel && (
        <p className="text-xs text-text-secondary mt-1">{changeLabel}</p>
      )}

      {sparklineData && (
        <div className="mt-3 h-12">
          <ResponsiveContainer width="100%" height="100%">
            <LineChart data={sparklineData.map((v, i) => ({ v, i }))}>
              <Line
                type="monotone"
                dataKey="v"
                stroke={isPositive ? 'var(--color-success)' : 'var(--color-error)'}
                strokeWidth={1.5}
                dot={false}
              />
            </LineChart>
          </ResponsiveContainer>
        </div>
      )}
    </div>
  );
}
```

---

## PART 3 — TRADINGVIEW LIGHTWEIGHT CHARTS (FINTECH)

### 3.1 OHLCV Candlestick Chart

```tsx
import { createChart, CandlestickSeries, HistogramSeries, ColorType } from 'lightweight-charts';
import { useEffect, useRef } from 'react';

interface OHLCV {
  time: number;
  open: number;
  high: number;
  low: number;
  close: number;
  volume: number;
}

function CandlestickChart({ data, symbol }: { data: OHLCV[]; symbol: string }) {
  const chartRef = useRef<HTMLDivElement>(null);
  const chartInstance = useRef<ReturnType<typeof createChart>>();

  useEffect(() => {
    if (!chartRef.current) return;

    const chart = createChart(chartRef.current, {
      layout: {
        background: { type: ColorType.Solid, color: 'transparent' },
        textColor: 'var(--color-text-secondary)',
        fontSize: 12,
      },
      grid: {
        vertLines: { color: 'var(--color-border)', style: 1 },
        horzLines: { color: 'var(--color-border)', style: 1 },
      },
      crosshair: {
        mode: 1,  // Normal crosshair
        vertLine: { color: 'var(--color-text-secondary)', width: 1, style: 2 },
        horzLine: { color: 'var(--color-text-secondary)', width: 1, style: 2 },
      },
      rightPriceScale: {
        borderColor: 'var(--color-border)',
        scaleMargins: { top: 0.1, bottom: 0.3 },
      },
      timeScale: {
        borderColor: 'var(--color-border)',
        timeVisible: true,
        secondsVisible: false,
      },
      handleScroll: { mouseWheel: true, pressedMouseMove: true },
      handleScale: { mouseWheel: true, pinch: true },
    });

    chartInstance.current = chart;

    // Candlestick series
    const candleSeries = chart.addSeries(CandlestickSeries, {
      upColor:   '#22c55e',
      downColor: '#ef4444',
      borderUpColor:   '#22c55e',
      borderDownColor: '#ef4444',
      wickUpColor:   '#22c55e',
      wickDownColor: '#ef4444',
    });
    candleSeries.setData(data);

    // Volume series (lower pane)
    const volumeSeries = chart.addSeries(HistogramSeries, {
      color: '#3b82f640',
      priceFormat: { type: 'volume' },
      priceScaleId: 'volume',
    });
    chart.priceScale('volume').applyOptions({
      scaleMargins: { top: 0.8, bottom: 0 },
    });
    volumeSeries.setData(
      data.map(d => ({
        time: d.time,
        value: d.volume,
        color: d.close >= d.open ? '#22c55e40' : '#ef444440',
      }))
    );

    // Fit content
    chart.timeScale().fitContent();

    // Resize observer
    const observer = new ResizeObserver(() => {
      chart.applyOptions({ width: chartRef.current!.clientWidth });
    });
    observer.observe(chartRef.current);

    return () => {
      observer.disconnect();
      chart.remove();
    };
  }, [data]);

  return (
    <div className="bg-surface border border-border rounded-xl overflow-hidden">
      <div className="flex items-center justify-between px-4 py-3 border-b border-border">
        <span className="font-semibold text-text-primary">{symbol}</span>
        <TimeframeSelector />
      </div>
      <div ref={chartRef} className="w-full h-[400px]" />
    </div>
  );
}
```

---

## PART 4 — MAPBOX GL JS

### 4.1 Choropleth Map (Galamsey Monitor Pattern)

```tsx
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';

mapboxgl.accessToken = process.env.NEXT_PUBLIC_MAPBOX_TOKEN!;

function GalamseyMap({ incidentData }: { incidentData: GeoJSON.FeatureCollection }) {
  const mapRef = useRef<HTMLDivElement>(null);
  const map = useRef<mapboxgl.Map>();

  useEffect(() => {
    if (!mapRef.current || map.current) return;

    map.current = new mapboxgl.Map({
      container: mapRef.current,
      style: 'mapbox://styles/mapbox/dark-v11',
      center: [-1.0232, 7.9465],   // Ghana
      zoom: 6.5,
    });

    map.current.on('load', () => {
      // Add incident data source
      map.current!.addSource('incidents', {
        type: 'geojson',
        data: incidentData,
        cluster: true,
        clusterMaxZoom: 12,
        clusterRadius: 50,
      });

      // Cluster circles
      map.current!.addLayer({
        id: 'clusters',
        type: 'circle',
        source: 'incidents',
        filter: ['has', 'point_count'],
        paint: {
          'circle-color': [
            'step', ['get', 'point_count'],
            '#22c55e',  10,   // < 10
            '#f59e0b',  30,   // 10-30
            '#ef4444',        // > 30
          ],
          'circle-radius': ['step', ['get', 'point_count'], 20, 10, 30, 30, 40],
          'circle-opacity': 0.85,
        },
      });

      // Cluster count labels
      map.current!.addLayer({
        id: 'cluster-count',
        type: 'symbol',
        source: 'incidents',
        filter: ['has', 'point_count'],
        layout: {
          'text-field': '{point_count_abbreviated}',
          'text-size': 12,
          'text-font': ['DIN Offc Pro Medium', 'Arial Unicode MS Bold'],
        },
        paint: { 'text-color': '#ffffff' },
      });

      // Individual incident markers
      map.current!.addLayer({
        id: 'unclustered-point',
        type: 'circle',
        source: 'incidents',
        filter: ['!', ['has', 'point_count']],
        paint: {
          'circle-color': '#ef4444',
          'circle-radius': 6,
          'circle-stroke-width': 2,
          'circle-stroke-color': '#fff',
        },
      });

      // Click handler for popups
      map.current!.on('click', 'unclustered-point', (e) => {
        const props = e.features![0].properties!;
        new mapboxgl.Popup()
          .setLngLat(e.lngLat)
          .setHTML(`
            <div class="p-2">
              <p class="font-semibold">${props.location}</p>
              <p class="text-sm text-gray-400">${props.date}</p>
              <p class="text-sm">${props.description}</p>
            </div>
          `)
          .addTo(map.current!);
      });

      map.current!.on('mouseenter', 'unclustered-point', () => {
        map.current!.getCanvas().style.cursor = 'pointer';
      });
      map.current!.on('mouseleave', 'unclustered-point', () => {
        map.current!.getCanvas().style.cursor = '';
      });
    });
  }, []);

  // Update data when incidentData changes
  useEffect(() => {
    if (map.current?.isStyleLoaded()) {
      (map.current.getSource('incidents') as mapboxgl.GeoJSONSource)
        .setData(incidentData);
    }
  }, [incidentData]);

  return <div ref={mapRef} className="w-full h-[500px] rounded-xl overflow-hidden" />;
}
```

---

## PART 5 — REAL-TIME DATA PATTERNS

### 5.1 WebSocket-Powered Live Chart

```tsx
function LiveMetricChart({ metric }: { metric: string }) {
  const [data, setData] = useState<DataPoint[]>([]);
  const wsRef = useRef<WebSocket>();

  useEffect(() => {
    const ws = new WebSocket(`wss://api.myapp.com/ws/metrics/${metric}`);
    wsRef.current = ws;

    ws.onmessage = (event) => {
      const point: DataPoint = JSON.parse(event.data);
      setData(prev => {
        const updated = [...prev, point];
        return updated.slice(-100);  // Keep last 100 points
      });
    };

    ws.onclose = (event) => {
      if (!event.wasClean) {
        // Reconnect with exponential backoff
        reconnect(metric);
      }
    };

    return () => ws.close();
  }, [metric]);

  // Throttle re-renders for high-frequency data
  const throttledData = useThrottle(data, 100);

  return (
    <ResponsiveContainer width="100%" height={200}>
      <AreaChart data={throttledData}>
        <defs>
          <linearGradient id="gradient" x1="0" y1="0" x2="0" y2="1">
            <stop offset="5%"  stopColor="var(--color-brand)" stopOpacity={0.3} />
            <stop offset="95%" stopColor="var(--color-brand)" stopOpacity={0} />
          </linearGradient>
        </defs>
        <Area
          type="monotone"
          dataKey="value"
          stroke="var(--color-brand)"
          strokeWidth={2}
          fill="url(#gradient)"
          dot={false}
          isAnimationActive={false}   // Disable for real-time (causes jitter)
        />
        <XAxis dataKey="timestamp" hide />
        <YAxis hide />
        <Tooltip content={<MinimalTooltip />} />
      </AreaChart>
    </ResponsiveContainer>
  );
}
```

---

## PART 6 — DATA TABLE WITH SORTING + FILTERING

```tsx
import {
  useReactTable, getCoreRowModel, getSortedRowModel,
  getFilteredRowModel, getPaginationRowModel,
  flexRender, type ColumnDef, type SortingState,
} from '@tanstack/react-table';

function DataTable<TData>({ columns, data }: { columns: ColumnDef<TData>[]; data: TData[] }) {
  const [sorting, setSorting] = useState<SortingState>([]);
  const [globalFilter, setGlobalFilter] = useState('');

  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
    state: { sorting, globalFilter },
    onSortingChange: setSorting,
    onGlobalFilterChange: setGlobalFilter,
    initialState: { pagination: { pageSize: 20 } },
  });

  return (
    <div className="space-y-3">
      <input
        value={globalFilter}
        onChange={e => setGlobalFilter(e.target.value)}
        placeholder="Search..."
        className="w-full max-w-sm px-3 py-2 text-sm border border-border rounded-lg bg-surface"
      />

      <div className="border border-border rounded-xl overflow-hidden">
        <table className="w-full text-sm">
          <thead>
            {table.getHeaderGroups().map(headerGroup => (
              <tr key={headerGroup.id} className="border-b border-border bg-bg-subtle">
                {headerGroup.headers.map(header => (
                  <th
                    key={header.id}
                    onClick={header.column.getToggleSortingHandler()}
                    className={cn(
                      'px-4 py-3 text-left font-medium text-text-secondary',
                      header.column.getCanSort() && 'cursor-pointer select-none hover:text-text-primary'
                    )}
                  >
                    <div className="flex items-center gap-1">
                      {flexRender(header.column.columnDef.header, header.getContext())}
                      {{ asc: ' ↑', desc: ' ↓' }[header.column.getIsSorted() as string] ?? ''}
                    </div>
                  </th>
                ))}
              </tr>
            ))}
          </thead>
          <tbody>
            {table.getRowModel().rows.map((row, i) => (
              <tr
                key={row.id}
                className={cn(
                  'border-b border-border last:border-0',
                  i % 2 === 0 ? 'bg-surface' : 'bg-bg-subtle'
                )}
              >
                {row.getVisibleCells().map(cell => (
                  <td key={cell.id} className="px-4 py-3 text-text-primary">
                    {flexRender(cell.column.columnDef.cell, cell.getContext())}
                  </td>
                ))}
              </tr>
            ))}
          </tbody>
        </table>
      </div>

      <TablePagination table={table} />
    </div>
  );
}
```

---

## PART 7 — COLOR THEORY FOR DATA

```
Sequential palettes (single concept, low → high):
  Blues:    #eff6ff → #1e40af  (safe for all audiences)
  Greens:   #f0fdf4 → #166534  (growth, positive)
  Reds:     #fff1f2 → #9f1239  (risk, negative)

Diverging palettes (deviation from center):
  Red–White–Blue:  for data with neutral midpoint (e.g. -50% to +50%)
  Use 7-step scales for smooth gradient

Categorical palettes (distinguishing groups):
  Max 8 colors — beyond that, use patterns or labels
  Ensure 3:1 contrast ratio between adjacent colors
  Colorblind-safe (no red–green as only differentiator):
    #3b82f6 (blue) + #f59e0b (amber) + #8b5cf6 (violet)
    + #06b6d4 (cyan) + #10b981 (emerald) + #f43f5e (rose)

Highlight pattern:
  All series: light gray
  Series of interest: brand color
  (Most effective for directing attention)
```

---

> **Core Principle**: The best visualization is the one that makes the answer obvious
> without explanation. If you need a legend to understand a chart, the chart has failed.
> Design for the insight, not the data.
