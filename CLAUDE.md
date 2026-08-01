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
7. **Polymarket — WORKING METHOD (verified Aug 2026)**: direct curl/WebFetch to gamma-api.polymarket.com are BLOCKED (container relay 403 + Polymarket edge blocks datacenter fetchers). The route that works: **Zapier MCP → "Webhooks by Zapier" → GET action** (`inspect_zapier_actions` for the schema, then `execute_zapier_write_action` with `tool_name: webhooks_by_zapier_get`; the request fires from Zapier's cloud). Key URLs: `https://gamma-api.polymarket.com/events` with query params `slug=what-price-will-bitcoin-hit-before-2027` (BTC yearly price ladder), or `closed=false&order=volume24hr&ascending=false&limit=N` (top markets), and `https://gamma-api.polymarket.com/public-search` with `q=<terms>` (find Fed/event markets; results nest under `results[0].events`). Responses >25k tokens land in a file — parse with python/jq (`outcomePrices`/`outcomes` are JSON-encoded strings). Ignore zero-volume duplicate events; use the one with real `volume24hr`. Fall back to web search only if the Zapier route fails, and label those odds with their article date.
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

### Newhedge MCP (mandatory layer — user instruction: "always use Newhedge")
- Every analysis must include the Newhedge on-chain/cycle layer. All 6 tools: `search_metrics`/`list_metrics`/`get_api_docs` (quota-free discovery) and `get_latest_value`/`get_metric`/`compare_metrics` (quota; 50k/month). Prefer one `compare_metrics` call with many specs + `limit: 2` for day-over-day.
- Core slug/metric pairs: `mvrv-z-score/mvrv_z`, `realized-price/realized_price`, `short-term-holder-realized-price/realized_price_sth`, `long-term-holder-realized-price/realized_price_lth`, `spent-output-profit-ratio/sopr`, `short-term-holder-sopr/sopr_sth`, `long-term-holder-sopr/sopr_lth`, `value-days-destroyed-multiple/vdd_multiple`, `percent-supply-in-profit/supply_in_profit_percent`, `net-unrealized-profit-loss/net_unrealized_profit_loss`, `funding-rate/bitcoin_funding_rate_{binance,okx,hyperliquid}`, `total-futures-open-interest/total_futures_open_interest`, `liquidations/bitcoin_liquidations_{binance,okx}_{long,short}`, `mayer-multiple/{mayer_multiple,btc_200_dma}`, `aviv-ratio/aviv_ratio`, `puell-multiple/puell_multiple`, `terminal-price/terminal_price`, `golden-ratio-multiplier/dma_350_btc`, `pi-cycle-top-indicator/dma_111_btc`, `sell-side-risk-ratio/sell_side_risk_ratio`, `choppiness-index/price_2w_choppiness_index`, `volatility-index/price_1w_volatility`, macro: `vix/vix_price_usd`, `dxy-correlation/dxy_price_usd`, `treasury-yields/treasury_30y_price_usd`, `gold-correlation/gold_price_usd`, `ethereum-correlation/btc_ethereum_correlation`, `altcoins-correlation/btc_altcoins_correlation`.
- Daily prints stamp at 00:00 UTC and publish ~00:09–00:30 UTC; always state the `as_of` date and flag stale stamps instead of guessing.
- The connector's OAuth flaps (tools vanish from ToolSearch, server shows "requires authentication"): the fix is the USER re-authorizing at claude.ai → Settings → Connectors → Newhedge. Retry ToolSearch after reconnect; never fabricate values while it is down.
- Newhedge has NO order-book/depth/wall/heatmap data (verified against the full catalog + docs): liquidation LEVELS come from user Coinglass screenshots, walls from the Crypto.com book. Newhedge liquidations metrics are daily per-exchange totals only.

### Environment notes
- The environment's network access has been opened by the user (Full/custom allowlist). Probe OKX, Coinglass, Polymarket, alternative.me, CoinGecko each run.
- Binance/Bybit REST: geo-blocked at the exchange edge (451) regardless of network policy — substitute OKX with identical metric definitions.
- LunarCrush MCP requires a paid subscription — skip unless the user upgrades.
