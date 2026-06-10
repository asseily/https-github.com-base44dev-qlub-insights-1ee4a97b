# Hyperliquid API access

This repo is configured with a Hyperliquid MCP server in [`.mcp.json`](.mcp.json)
(`@mektigboy/server-hyperliquid`, run via `npx`). It exposes public market-data
tools backed by `https://api.hyperliquid.xyz`:

- `get_all_mids` — mid prices for all coins
- `get_candle_snapshot` — OHLCV candles for a coin/interval
- `get_l2_book` — level-2 order book for a coin

## Required: allow the API domain (one-time, manual)

Claude Code on the web sandboxes block outbound traffic to hosts outside the
environment's allowlist, and this cannot be changed from inside a session.
To enable the API:

1. Go to [claude.ai/code](https://claude.ai/code).
2. Click the cloud icon showing the current environment's name, hover over the
   environment, and click the settings icon.
3. Set **Network access** to **Custom**.
4. In **Allowed domains**, add:
   ```
   api.hyperliquid.xyz
   ```
5. Check **Also include default list of common package managers** (needed so
   `npx` can still install the MCP server from npm).
6. Save, then start a **new** session (existing containers keep the old policy).

After that, both the MCP tools above and direct `curl` calls work, e.g.:

```bash
curl -s -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type":"allMids"}'
```

## Authenticated / trading access

The bundled server is read-only public data. For order placement or account
endpoints you'd need a server that signs requests with an API wallet key —
e.g. one built on the [`@nktkas/hyperliquid`](https://www.npmjs.com/package/@nktkas/hyperliquid)
SDK — with the private key supplied via an environment variable in the
environment settings (never committed to the repo).

Docs: <https://code.claude.com/docs/en/claude-code-on-the-web> (Network access section)
