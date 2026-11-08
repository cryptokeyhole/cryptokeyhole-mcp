# CryptoKeyhole MCP Server

A hosted, read-only [Model Context Protocol](https://modelcontextprotocol.io) server that gives AI assistants and agents direct access to [CryptoKeyhole](https://cryptokeyhole.com) on-chain and market metrics for Bitcoin, Ethereum and a growing list of coins.

```
https://cryptokeyhole.com/api/v1/mcp
```

- **Remote, nothing to install.** Streamable HTTP transport; point your MCP client at the URL above.
- **No account, no API key.** The server answers without credentials and serves everything the CryptoKeyhole Free plan includes.
- **Our own data.** Bitcoin and Ethereum metrics are calculated from CryptoKeyhole's own full nodes; market series come from the market data CryptoKeyhole collects. Each result carries its source, method and attribution.
- **Answers with charts.** The question tools answer common market questions in one call, with the time of the reading and a link to the page the figures come from; clients that show MCP Apps also get an interactive chart.
- **Exact values.** Numbers arrive as decimal strings with their finality, source clocks and revision, and missing data is reported as a gap, never as zero.
- **Listed in the official MCP Registry** as [`com.cryptokeyhole/analytics`](https://registry.modelcontextprotocol.io/v0.1/servers?search=com.cryptokeyhole).

Documentation on the site: [cryptokeyhole.com/mcp/](https://cryptokeyhole.com/mcp/)

## Contents

- [Tools](#tools)
- [Setup](#setup)
  - [Claude Code](#claude-code)
  - [Claude Desktop and claude.ai](#claude-desktop-and-claudeai)
  - [Cursor](#cursor)
  - [VS Code](#vs-code)
  - [Windsurf](#windsurf)
  - [Gemini CLI](#gemini-cli)
  - [Antigravity](#antigravity)
  - [Any Streamable HTTP client](#any-streamable-http-client)
- [Example prompts](#example-prompts)
- [What the free server covers](#what-the-free-server-covers)
- [Reading the results](#reading-the-results)
- [Other ways to get the data](#other-ways-to-get-the-data)
- [Files in this repository](#files-in-this-repository)

## Tools

All tools are read-only and idempotent.

### Question tools

Each one answers a common question in one call: a short answer with the time of the reading, a link to the page on cryptokeyhole.com and, in clients that show MCP Apps, an interactive chart.

| Tool | What it answers |
|---|---|
| `get_liquidation_map` | Where leveraged Bitcoin or Ethereum futures positions would be liquidated right now: the largest long and short clusters around the price, the totals within 5% and the map's measured accuracy. Redrawn every five minutes. |
| `get_derivatives_snapshot` | Bitcoin or Ethereum futures liquidations in the last 24 hours, the funding rate averaged across venues by open interest, open interest with its 24-hour change, and the next options expiry's max pain. |
| `get_price_history` | A coin's daily close on any UTC day, or its change between two days with the highest and lowest close in between. |
| `get_eth_btc_ratio` | The ETH/BTC ratio now with its 30-day change, 200-day average and all-time high close, or its change between two days. |
| `get_support_levels` | Bitcoin's price against the levels traders watch: the 200-day, 200-week and 1-year averages, the bull market support band, the Pi Cycle Bottom line, the realized price and the power-law lines. |
| `get_exchange_reserves` | Bitcoin, ether and stablecoins at each exchange that publishes its addresses, read on chain, with net flows over 24 hours and 7 days and flags such as unusual outflows. |
| `get_etf_flows` | Daily net flows of the US spot ETFs of one coin, or of every coin together, from the issuers' own daily files. |

### Data tools

Any metric on the site, by its exact id:

| Tool | What it does |
|---|---|
| `get_assets_list` | Lists the assets CryptoKeyhole covers, with their ids (`BTC`, `ETH`, `LINK` and more). |
| `get_asset_metrics` | Finds an asset's metrics by words (for example "puell" or "exchange balance") or by category, with each metric's exact id, unit, intervals, history and the plan its data needs. |
| `get_metrics_list` | The same search and rows, for BTC by default. |
| `get_metric_metadata` | Returns a metric's definition, supported intervals, method, coverage and source limitations. |
| `fetch_metric` | Returns the exact observations of one metric for a time range, with source clocks, revision, finality and gaps. Optional SMA, EMA or median smoothing. |
| `fetch_bulk_metrics` | Reads several metric selections in one call. Each keeps its own provenance and revision; together they are not one atomic snapshot. |

`fetch_metric` arguments:

| Argument | Meaning |
|---|---|
| `metric` | Metric id, as in the chart's address. For example `ckh.MayerMultiple` from `https://cryptokeyhole.com/charts/btc/?metric=ckh.MayerMultiple`. Required. |
| `asset` | Asset id, `BTC` by default. |
| `interval` | `24h` by default; also `1w`, `1M` and others where the metric offers them. |
| `start`, `end` | UTC Unix seconds. The start is inclusive and the end exclusive. |
| `sma`, `smoothing` | Smoothing window in observations (0 for none) and its method: `sma`, `ema` or `median`. |
| `publication` | A retained publication id, for the metrics that keep them. Leave it out for current data. |

## Setup

The endpoint is the same for every client: `https://cryptokeyhole.com/api/v1/mcp`. Don't add an `Authorization` header or any other credentials; the server refuses requests that carry them.

### Claude Code

```bash
claude mcp add --transport http cryptokeyhole https://cryptokeyhole.com/api/v1/mcp
```

Or in a project's `.mcp.json`:

```json
{
  "mcpServers": {
    "cryptokeyhole": {
      "type": "http",
      "url": "https://cryptokeyhole.com/api/v1/mcp"
    }
  }
}
```

### Claude Desktop and claude.ai

CryptoKeyhole is in Claude's connector directory: [claude.ai/directory/cryptokeyhole](https://claude.ai/directory/cryptokeyhole).

Or open **Settings > Connectors > Add custom connector**, name it `CryptoKeyhole` and paste `https://cryptokeyhole.com/api/v1/mcp` as the URL. No OAuth settings are needed.

If you prefer the `claude_desktop_config.json` file, which starts local processes, bridge to the remote server with [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) (needs Node.js):

```json
{
  "mcpServers": {
    "cryptokeyhole": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://cryptokeyhole.com/api/v1/mcp"]
    }
  }
}
```

### Cursor

`~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (one project):

```json
{
  "mcpServers": {
    "cryptokeyhole": {
      "url": "https://cryptokeyhole.com/api/v1/mcp"
    }
  }
}
```

### VS Code

`.vscode/mcp.json` in your workspace:

```json
{
  "servers": {
    "cryptokeyhole": {
      "type": "http",
      "url": "https://cryptokeyhole.com/api/v1/mcp"
    }
  }
}
```

Or from a terminal:

```bash
code --add-mcp '{"name":"cryptokeyhole","type":"http","url":"https://cryptokeyhole.com/api/v1/mcp"}'
```

### Windsurf

`~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "cryptokeyhole": {
      "serverUrl": "https://cryptokeyhole.com/api/v1/mcp"
    }
  }
}
```

### Gemini CLI

Install it as an extension from this repository:

```bash
gemini extensions install https://github.com/cryptokeyhole/cryptokeyhole-mcp
```

Or add it to `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "cryptokeyhole": {
      "httpUrl": "https://cryptokeyhole.com/api/v1/mcp"
    }
  }
}
```

### Antigravity

Install it as a plugin from this repository:

```bash
agy plugin install https://github.com/cryptokeyhole/cryptokeyhole-mcp
```

Or add it to `mcp_config.json`:

```json
{
  "mcpServers": {
    "cryptokeyhole": {
      "serverUrl": "https://cryptokeyhole.com/api/v1/mcp"
    }
  }
}
```

### Any Streamable HTTP client

The server is stateless: it issues no session id, and every JSON-RPC request is a standalone `POST`. Clients that run the usual `initialize` handshake first work as normal. To try it from a shell:

```bash
# List the tools
curl -s https://cryptokeyhole.com/api/v1/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -H 'MCP-Protocol-Version: 2025-06-18' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'

# Daily Bitcoin Mayer Multiple from 1 October 2026 (Unix seconds, UTC)
curl -s https://cryptokeyhole.com/api/v1/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -H 'MCP-Protocol-Version: 2025-06-18' \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"fetch_metric","arguments":{"asset":"BTC","metric":"ckh.MayerMultiple","interval":"24h","start":1790812800}}}'
```

## Example prompts

Once the server is connected, ask in plain language:

- "Where are the Bitcoin liquidation clusters right now?"
- "How much ETH got liquidated in the last 24 hours, and what are funding and open interest doing?"
- "Have the US spot bitcoin ETFs been buying or selling this week?"
- "Is Bitcoin above its 200-week moving average? What are the nearest support levels?"
- "What is Bitcoin's Mayer Multiple today, and how often has it been higher over the past four years?"
- "Fetch Bitcoin's daily hash rate for 2024 and tell me which month grew the most."
- "How far is Bitcoin's price below its all-time high right now, and how does that compare with past cycles?"
- "Show the share of ETH that is staked, month by month since 2023."
- "Which Ethereum metrics does CryptoKeyhole have about stablecoins? Describe how the supply one is calculated."
- "Is the latest hour of Bitcoin data final yet? If not, why not?"

## What the free server covers

The MCP server reads what the CryptoKeyhole Free plan includes: the Free charts, current and with full history, at daily, weekly, monthly and yearly resolution. A metric that belongs to a paid plan answers with a short message naming the plan that includes it, so your assistant can tell the difference between "not free" and "no data".

The metric ids are the same as on the site and in the [free REST API](https://cryptokeyhole.com/data-access/), so anything you find through the assistant can be opened as a chart at `https://cryptokeyhole.com/charts/<asset>/?metric=<id>`.

## Reading the results

- **Exact decimals.** Values are decimal strings so no precision is lost. Ask your assistant not to round unless you want it to.
- **Gaps, not zeros.** A missing observation is reported as a gap.
- **Time.** Ranges use UTC Unix seconds, with an inclusive start and an exclusive end.
- **Finality.** Every observation says whether it is final, partial or provisional. Bitcoin's latest hour stays delayed until the chain's median time past seals it; Ethereum-based series publish once Ethereum has finalised the hour; market series publish once each hour has closed. Partial or provisional values can still change.
- **Held metrics.** A metric that is not yet published is reported as held, with the reason, rather than with substitute values.
- **Not an atomic snapshot.** Bulk reads are taken one after another.
- **Attribution.** Results carry each series' source and attribution. Keep them when you reuse the data; see the [terms of service](https://cryptokeyhole.com/terms/).

## Other ways to get the data

- **Charts and dashboards:** [cryptokeyhole.com](https://cryptokeyhole.com)
- **Indicator guides:** what each indicator measures and how to read it, at [cryptokeyhole.com/learn/](https://cryptokeyhole.com/learn/), with a plain-text reference in [cryptokeyhole/onchain-indicators](https://github.com/cryptokeyhole/onchain-indicators)
- **Free REST API, Google Sheets, Excel and Python:** [cryptokeyhole.com/data-access/](https://cryptokeyhole.com/data-access/)
- **Free chart embeds** for blogs, newsletters and forums: [cryptokeyhole.com/embed/](https://cryptokeyhole.com/embed/)

## Files in this repository

- [`gemini-extension.json`](gemini-extension.json): the Gemini CLI extension.
- [`plugin.json`](plugin.json) and [`mcp_config.json`](mcp_config.json): the Antigravity plugin, with the icon in [`assets/`](assets/).
- [`server.json`](server.json) mirrors the server's entry in the official MCP Registry:

```bash
curl -s 'https://registry.modelcontextprotocol.io/v0.1/servers?search=com.cryptokeyhole'
```

## Licence

The contents of this repository (documentation and configuration examples) are under the [MIT licence](LICENSE). Data served by the MCP server is covered by CryptoKeyhole's [terms of service](https://cryptokeyhole.com/terms/).

Questions or problems: open an [issue](https://github.com/cryptokeyhole/cryptokeyhole-mcp/issues).
