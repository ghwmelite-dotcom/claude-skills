# ETF (Exchange-Traded Fund) Reference

## ETF Fundamentals

### What Is an ETF?
An ETF is a basket of securities that trades on an exchange like a single stock.
- Combines diversification of mutual funds with intraday tradability of stocks
- Creation/Redemption mechanism keeps price close to NAV (Net Asset Value)
- **Authorized Participants (APs)** arbitrage away premiums/discounts

### ETF vs Mutual Fund vs Index Fund
| Feature | ETF | Mutual Fund | Index Fund |
|---|---|---|---|
| Trading | Intraday | End-of-day NAV | End-of-day NAV |
| Expense Ratio | Low (0.03-0.75%) | High (0.5-1.5%) | Low (0.03-0.2%) |
| Tax Efficiency | High (in-kind creation) | Lower | Moderate |
| Min Investment | 1 share (~$1 with fractional) | Often $1,000+ | Varies |
| Transparency | Daily holdings disclosed | Monthly/quarterly | Daily |

---

## ETF Types & Categories

### Broad Market / Core ETFs
| ETF | Index Tracked | Expense Ratio | AUM | Notes |
|---|---|---|---|---|
| **SPY** | S&P 500 | 0.0945% | ~$500B | Most liquid ETF; options market huge |
| **IVV** | S&P 500 | 0.03% | ~$400B | Cheaper SPY alternative |
| **VOO** | S&P 500 | 0.03% | ~$400B | Vanguard flagship |
| **QQQ** | Nasdaq 100 | 0.20% | ~$200B | Most popular tech ETF |
| **QQQM** | Nasdaq 100 | 0.15% | ~$30B | Cheaper QQQ for long-term holders |
| **VTI** | Total US Market | 0.03% | ~$350B | Broadest US coverage |
| **VT** | Global Total Market | 0.07% | ~$35B | All-world stocks |
| **IWM** | Russell 2000 | 0.19% | ~$60B | Small-cap; higher beta |
| **DIA** | Dow Jones | 0.16% | ~$30B | Less popular; Dow tracks |

### International ETFs
| ETF | Exposure | Notes |
|---|---|---|
| **EFA** | Developed markets ex-US | Europe, Japan, Australia |
| **EEM** | Emerging markets | China, India, Brazil, S. Korea |
| **VWO** | Emerging markets | Vanguard; cheaper than EEM |
| **IEMG** | Emerging markets | iShares; popular |
| **FXI** | China large-cap | Alibaba, Tencent heavy |
| **INDA** | India | Growing strategic importance |
| **EWJ** | Japan | JPY exposure risk |
| **AGG** | Africa exposure | Limited dedicated Africa ETFs |

### Sector ETFs (SPDR Select Sectors)
| ETF | Sector |
|---|---|
| XLK | Technology |
| XLF | Financials |
| XLE | Energy |
| XLV | Healthcare |
| XLY | Consumer Discretionary |
| XLP | Consumer Staples |
| XLI | Industrials |
| XLB | Materials |
| XLU | Utilities |
| XLRE | Real Estate |
| XLC | Communication Services |

### Bond ETFs
| ETF | Type | Duration | Yield Context |
|---|---|---|---|
| **BND** | Total US Bond Market | Intermediate | Broad exposure |
| **AGG** | US Aggregate Bond | Intermediate | Core fixed income |
| **TLT** | 20+ Year Treasury | Long | Rate-sensitive; inverse to yields |
| **IEF** | 7-10 Year Treasury | Intermediate | Moderate duration |
| **SHY** | 1-3 Year Treasury | Short | Near-cash; minimal rate risk |
| **HYG** | High-Yield Bonds | Intermediate | Junk bonds; credit risk |
| **LQD** | Investment-Grade Corp | Intermediate | Corporate credit |
| **TIPS** | Inflation-Protected | Intermediate | Real yield; inflation hedge |
| **EMB** | Emerging Market Bonds | Intermediate | EM credit + FX risk |

**Duration rule**: 10-yr duration bond ETF falls ~10% for every 1% rise in rates.
- TLT (duration ~17): Lost ~40% in 2022 rate hike cycle

---

## Commodity & Alternatives ETFs

### Gold & Metals
| ETF | Type | Notes |
|---|---|---|
| **GLD** | Physical Gold | Most liquid; 0.40% ER |
| **IAU** | Physical Gold | Cheaper (0.25%); iShares |
| **SGOL** | Physical Gold | Swiss-vaulted gold |
| **SLV** | Physical Silver | Silver equivalent of GLD |
| **GDX** | Gold Miners | GDXJ for junior miners; leverage to gold |
| **GDXJ** | Junior Gold Miners | Higher beta to gold |
| **PPLT** | Physical Platinum | Platinum exposure |

### Energy & Broad Commodity
| ETF | Exposure |
|---|---|
| **USO** | WTI Crude Oil (futures-based; contango drag!) |
| **DBO** | Oil (optimized roll; better for long-term) |
| **XLE** | Energy companies (avoids contango) |
| **DJP** | Broad commodities index |
| **GSG** | iShares S&P GSCI Commodity |

**Warning on futures-based commodity ETFs**: Contango drag erodes returns over time.
Prefer equity-based (XLE) over futures-based (USO) for long-term commodity exposure.

---

## Leveraged & Inverse ETFs

### How They Work
- 2x/3x daily rebalancing — **not 2x/3x over time** (volatility decay)
- Suitable only for short-term tactical use (days to weeks max)
- Volatility decay destroys long-term value in choppy markets

### Popular Leveraged ETFs
| ETF | Exposure | Leverage |
|---|---|---|
| TQQQ | Nasdaq 100 | 3x Daily |
| SQQQ | Inverse Nasdaq 100 | -3x Daily |
| UPRO | S&P 500 | 3x Daily |
| SPXU | Inverse S&P 500 | -3x Daily |
| SOXL | Semiconductors | 3x Daily |
| TNA | Small-Cap (IWM) | 3x Daily |
| UVXY | VIX Futures | ~1.5x; extreme decay |

**Rule**: Never hold leveraged ETFs as long-term positions. Use only for tactical short-term momentum plays with tight stops.

---

## ETF Analysis Framework

### Evaluating an ETF
```
1. EXPENSE RATIO: Lower is better; even 0.5% compounds significantly
   - Rule: For same exposure, always choose lower ER
   
2. LIQUIDITY (AUM + Volume):
   - AUM >$500M = safe
   - Daily volume >$10M = tight bid/ask spreads
   - Small ETFs: Risk of closure or wide spreads
   
3. TRACKING ERROR:
   - Should be <0.10% for major index ETFs
   - Physical replication < synthetic (swap-based) for tracking
   
4. HOLDINGS & CONCENTRATION:
   - Top 10 holdings as % of fund (>40% = concentrated)
   - For sector ETFs: How diversified within sector?
   
5. INDEX CONSTRUCTION:
   - Market-cap weighted (most common) vs equal weight vs fundamentals
   - Equal weight (RSP for S&P 500): Less FAANG concentration
   
6. DIVIDEND TREATMENT:
   - Accumulating (reinvests dividends) vs Distributing (pays out)
   - Accumulating = better for long-term compounding; distributing = income
   
7. DOMICILE (for non-US investors):
   - US-domiciled ETFs (SPY, QQQ): 30% withholding tax on dividends for non-US
   - Ireland-domiciled ETFs (CSPX, IUSA on LSE): 15% treaty rate; better for non-US
   - Ghanaian investors: Ireland-domiciled ETFs are typically more tax-efficient
```

---

## ETF Portfolio Construction

### Classic Core-Satellite Model
```
Core (60-80%): Broad, low-cost index ETFs
├── US Equity: VOO or VTI (40%)
├── International: VEA + VWO (20%)
└── Bonds: BND or AGG (20%)

Satellite (20-40%): Thematic/tactical tilts
├── Sector bet: XLK or ARKK (tech momentum)
├── Factor: QQQM (growth) or VBR (small-cap value)
├── Commodity: GLD (inflation hedge)
└── Income: JEPI (covered calls) or SCHD (dividend)
```

### ETF Strategy by Objective
**Aggressive Growth**: QQQ (40%) + VTI (30%) + IEMG (20%) + GLD (10%)
**Balanced**: VOO (40%) + BND (30%) + VEA (20%) + GLD (10%)
**Income**: SCHD (30%) + JEPI (25%) + SPLV (20%) + HYG (15%) + TIPS (10%)
**Inflation Hedge**: GLD (25%) + TIP (25%) + DJP (20%) + XLE (15%) + VNQ (15%)
**All-Weather**: VTI (30%) + TLT (40%) + GLD (15%) + DJP (15%)

---

## Options on ETFs (Advanced)

- **SPY, QQQ, IWM** have the most liquid options markets globally
- **0DTE (0 days to expiration)**: Same-day options on SPY — massive volume, high risk
- **LEAPS**: Long-dated options (1-2yr) for leveraged long/short with defined risk
- **Covered Calls**: Buy ETF + sell calls for income; JEPI does this systematically
- **Protective Puts**: Buy puts on SPY as portfolio insurance

---

## Africa/Ghana-Relevant ETF Investing Notes

- **Access method**: Interactive Brokers (best for African residents), TD Ameritrade, or local brokers with US market access
- **Regulated exchanges**: NSE (Nigeria), GSE (Ghana Stock Exchange) have limited ETF products
- **GSE**: No domestic ETFs as of 2024; investors use offshore brokers
- **Currency risk**: GHS/USD exchange rate erodes or amplifies returns
- **Withholding tax**: 30% for US-domiciled ETF dividends; use Ireland-domiciled on LSE where possible (CSPX, IWDA, VWRA)
- **Best Africa-focused ETFs**: AFK (VanEck Africa), EZA (South Africa), NGE (Nigeria) — limited liquidity
