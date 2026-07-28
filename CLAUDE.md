# CLAUDE.md

## Market Analysis Protocol (standing instruction)

Whenever the user asks for market analysis, price targets, or trade entries for BTC or ANY asset ("analyze BTC", "give me entries", "run the sweep", "check the market", or similar), ALWAYS run the FULL sweep below before answering. Never give entries from a single data source or a single timeframe.

### User context
- Timezone: Beirut (UTC+3). "3am my time" = 00:00 UTC.
- Default account assumption for sizing: $10,000, 1% risk per trade. Halve size within 24h of a Tier-1 event (FOMC, CPI, NFP).
- The user often pastes Coinglass screenshots — read them directly and fold the liquidation levels into the analysis.

### Skills to load first
- Prefer the `crypto-signal-monitor` or `max` skill if available in the session — they implement most of this protocol.
- The MAX skill's bundled scripts (`ta_engine.py`, `price_forecast.py`, `polymarket_scanner.py`) do TA, Monte Carlo, and Polymarket scanning — use them instead of rebuilding.

### Data to collect (all of it, in parallel where possible)
1. **All timeframes**: 15m, 1h, 4h, 1D candles — map swing highs/lows, market stage, and where price sits in each range.
2. **Indicators** computed from those candles: RSI(14), MACD, Bollinger squeeze/expansion, volume vs average.
3. **Order book** (50 levels): bid/ask imbalance, walls, iceberg ladders.
4. **Live trade tape** (150 trades): aggressive buy vs sell volume split, absorption check.
5. **Derivatives** — funding rate + trend, open interest + 24h trend, global AND top-trader long/short ratios, taker buy/sell ratio. Use **OKX** as primary (Binance and Bybit geo-block the cloud VMs with HTTP 451 — do not burn time on them; a quick probe is fine).
6. **Coinglass liquidation heatmap** via headless Chromium screenshots (chromium at /opt/pw-browsers): clusters above/below price with brightness, funding heatmap, 24h liquidation totals. If network-blocked, ask the user for a screenshot instead. NEVER fabricate heatmap numbers.
7. **Polymarket** (gamma-api.polymarket.com): odds on relevant Fed/crypto markets. Fall back to web search if blocked.
8. **Sentiment**: Fear & Greed (api.alternative.me), options data if reachable (max pain, DVOL, put/call walls).
9. **Monte Carlo forecast** sized to the user's stated session window (default: next 12h).
10. **Web sweep**: macro calendar for the next 48h (FOMC/CPI/NFP), geopolitical shocks, analyst levels.

### Output format (always)
- Numbered entries **in firing order** (which triggers first, and why — distance + tape direction + liquidity magnets).
- Each entry: entry zone, stop (with the structural reason), T1/T2 targets with R:R, position size math, and explicit invalidation condition.
- State the data snapshot timestamp. Flag every source that failed or is stale — report gaps honestly instead of inventing numbers.
- Flag conflicting signals explicitly and reduce conviction accordingly.
- If there is no edge: say WAIT clearly. Cash is a position.
- End with the standing risk reminders: scenario analysis not financial advice; flat before Tier-1 macro events; thin weekend liquidity = smaller size.

### Environment notes
- The environment's network access has been opened by the user (Full/custom allowlist). Probe OKX, Coinglass, Polymarket, alternative.me, CoinGecko each run.
- Binance/Bybit REST: geo-blocked at the exchange edge (451) regardless of network policy — substitute OKX with identical metric definitions.
- LunarCrush MCP requires a paid subscription — skip unless the user upgrades.
