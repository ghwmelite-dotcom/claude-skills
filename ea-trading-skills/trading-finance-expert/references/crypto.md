# Cryptocurrency Reference

## Market Structure

### Tier 1 — Blue Chips
- **Bitcoin (BTC)**: Store of value narrative; 21M cap; halving cycles (~4yr); dominance index (BTC.D)
- **Ethereum (ETH)**: Smart contract platform; EIP-1559 burn mechanism; staking yield; L2 ecosystem
- **BNB**: Binance ecosystem token; burn mechanism; BSC gas fee

### Tier 2 — Large Caps
Solana (SOL), XRP, Cardano (ADA), Avalanche (AVAX), Polkadot (DOT), Chainlink (LINK), Litecoin (LTC)

### Tier 3 — Mid/Small Caps
DeFi tokens (UNI, AAVE, COMP, CRV), L2 tokens (ARB, OP, MATIC/POL), Meme coins (DOGE, SHIB, PEPE)

### Stablecoins
- **USDT** (Tether): Largest; centralized; some reserve transparency concerns
- **USDC** (Circle): Regulated; US bank-backed; preferred by institutions
- **DAI/sDAI**: Decentralized; over-collateralized via MakerDAO
- **FDUSD, TUSD, PYUSD**: Exchange-native or fintech-backed

---

## Bitcoin Cycle Analysis

### Halving Cycle (Approx. 4-Year Pattern)
```
Halving → 6-12 months accumulation → Parabolic bull run → ATH → Bear market → Accumulation
Halvings: 2012, 2016, 2020, 2024 (April)
Post-halving ATH timeline: ~12-18 months after halving historically
```

### Key Bitcoin Metrics
- **Stock-to-Flow (S2F)**: Scarcity model (controversial but widely tracked)
- **MVRV Z-Score**: Market cap vs Realized cap; overheated >7, bottoming <0
- **NUPL (Net Unrealized Profit/Loss)**: Sentiment; euphoria zone >0.75
- **Realized Price**: Average price all BTC last moved; key support
- **Exchange Reserves**: Declining = accumulation (bullish); rising = selling pressure
- **Miner Revenue / Hash Rate**: Hash rate ATH often precedes price ATH

### Dominance (BTC.D)
- Rising BTC.D = Bitcoin outperforming alts (risk-off within crypto)
- Falling BTC.D = Altcoin season (risk-on); usually follows BTC ATH confirmation
- ETH.D rising separately = ETH decoupling, often DeFi-driven

---

## On-Chain Analytics

### Key Metrics to Track
| Metric | Signal | Where |
|---|---|---|
| Exchange Inflow/Outflow | High inflow = selling pressure | Glassnode, CryptoQuant |
| Whale Transactions >$1M | Spikes often precede moves | Whale Alert, Glassnode |
| Open Interest (OI) | Rising OI + rising price = healthy; OI peaks = liquidation risk | Coinglass |
| Funding Rate | Positive = longs paying shorts (crowded longs = correction risk) | Coinglass, Bybit |
| Liquidation Heatmap | Clusters of liquidations attract price (magnet effect) | Coinglass |
| Stablecoin Supply | Rising USDC/USDT supply = more buying power entering | Glassnode |
| Active Addresses | Rising with price = real adoption; divergence = warning | Glassnode |
| Long/Short Ratio | Extreme readings are contrarian signals | Coinglass |

---

## DeFi (Decentralized Finance)

### Core Protocols
- **Uniswap / Curve / Balancer**: AMM DEXes; impermanent loss is key risk
- **Aave / Compound**: Lending/borrowing; utilization rate drives yield
- **MakerDAO**: DAI stablecoin; CDP vaults; MATIC/ETH collateral
- **Lido / Rocket Pool**: Liquid staking (stETH, rETH); staking yield + liquidity
- **Pendle Finance**: Yield tokenization; fixed vs variable yield splitting
- **GMX / dYdX**: Perpetual DEXes; GLP liquidity provision

### DeFi Metrics
- **TVL (Total Value Locked)**: Health indicator; rising TVL = capital inflow
- **APY vs APR**: APY compounds; compare correctly
- **Impermanent Loss (IL)**: Risk when providing liquidity to volatile pairs
- **Protocol Revenue**: Sustainable yield vs inflationary emissions
- **Audit Status**: Always check; unaudited protocols = high rug risk

### Yield Farming Safety Framework
```
Before entering any DeFi position:
1. Is protocol audited? (Certik, Trail of Bits, OpenZeppelin)
2. Is it a known team or anon? (Anon + unaudited = high risk)
3. How old is the protocol? (<6 months = high risk)
4. Is yield from fees (sustainable) or token emissions (temporary)?
5. IL risk: Stablecoin pairs = lowest; volatile/stable = medium; volatile/volatile = high
```

---

## Crypto Market Correlations

- **BTC → Alts**: BTC dumps hard → alts dump harder (beta effect)
- **BTC → SPX**: During risk-off macros, correlation rises toward 0.7+
- **BTC → DXY**: Negative correlation during macro-driven markets
- **BTC.D → Alts**: Inverse relationship (alt season when BTC.D falls)
- **ETH/BTC ratio**: Rising = ETH outperforming; key indicator of DeFi/NFT cycles

---

## Crypto Trading Specifics

### Spot vs Perpetual Futures
- **Spot**: Own the asset; no liquidation risk; earn staking/lending yield
- **Perpetual Futures**: Leverage up to 100x; funding rates; liquidation risk
- **Mark Price vs Last Price**: Use mark price for liquidation to prevent manipulation

### Key Exchanges
- **Tier 1 CEX**: Binance, Coinbase, OKX, Bybit, Kraken
- **DEX**: Uniswap (ETH), Raydium (Solana), PancakeSwap (BSC)
- **Derivatives**: Bybit, Binance Futures, dYdX, GMX

### Crypto-Specific Risk Events
- **Exchange hacks/insolvency**: Mt.Gox, FTX — systemic risk
- **Regulatory actions**: SEC vs Binance/Coinbase, Gensler era
- **Smart contract exploits**: Flash loan attacks, reentrancy bugs
- **Stablecoin depegs**: UST/LUNA collapse 2022 case study
- **Whale sell walls**: Identifiable on order book depth charts

### Technical Analysis Notes for Crypto
- **Weekly/Monthly closes matter** more in crypto than forex
- **Crypto trades 24/7** — no gaps, but Sunday closes have historical significance
- **Halving levels as S/R**: Previous halving price is often mid-cycle support
- **Psychological levels**: $10K, $20K, $30K, $50K, $100K for BTC

---

## XRP & Cross-Border Payments Context (Relevant for Ozzy's Apex10 project)

- **ODL (On-Demand Liquidity)**: XRP used as bridge currency; RippleNet corridors
- **Key corridors**: USD→MXN (Bitso), USD→PHP (Coins.ph), USD→NGN (emerging)
- **Escrow releases**: 1B XRP monthly from escrow; unsold returned; ~500M/month effective
- **Whale wallets**: Top 100 addresses hold ~80% supply; monitor for large movements
- **SEC lawsuit resolution (2024)**: XRP ruled not a security for retail sales; opened ETF possibilities
- **Price catalysts**: ETF approval, new ODL corridors, CBDC partnerships, regulatory clarity

---

## Crypto Regulation Landscape

| Jurisdiction | Status |
|---|---|
| USA | SEC/CFTC ongoing; spot BTC ETF approved Jan 2024; ETH ETF approved May 2024 |
| EU | MiCA regulation in force 2024 — stablecoin rules, CASP licensing |
| Ghana/Africa | Generally permissive but no formal framework; BOG cautious on crypto payments |
| El Salvador | BTC legal tender; Chivo wallet |
| UAE/Singapore | Crypto-friendly with licensing frameworks |
