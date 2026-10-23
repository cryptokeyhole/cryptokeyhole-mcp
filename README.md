# CryptoKeyhole MCP Server

A hosted, read-only [Model Context Protocol](https://modelcontextprotocol.io) server that gives AI assistants and agents direct access to [CryptoKeyhole](https://cryptokeyhole.com) on-chain and market metrics for Bitcoin, Ethereum and a growing list of coins.

```
https://cryptokeyhole.com/api/v1/mcp
```

- **Remote, nothing to install.** Streamable HTTP transport; point your MCP client at the URL above.
- **No account, no API key.** The server answers without credentials and serves everything the CryptoKeyhole Free plan includes.
- **Our own data.** Bitcoin and Ethereum metrics are calculated from CryptoKeyhole's own full nodes; market series come from the market data CryptoKeyhole collects. Each result carries its source, method and attribution.
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
  - [Any Streamable HTTP client](#any-streamable-http-client)
- [Example prompts](#example-prompts)
- [What the free server covers](#what-the-free-server-covers)
- [Reading the results](#reading-the-results)
- [Other ways to get the data](#other-ways-to-get-the-data)
- [Registry entry](#registry-entry)

## Tools

All tools are read-only and idempotent.

| Tool | What it does |
|---|---|
| `get_assets_list` | Lists the assets CryptoKeyhole covers, with their ids (`BTC`, `ETH`, `LINK` and more). |
| `get_metrics_list` | Finds an asset's metrics by category, including metrics that are delayed or held, with cursor paging. |
| `get_asset_metrics` | Lists the available and held metrics of one asset. |
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

Open **Settings > Connectors > Add custom connector**, name it `CryptoKeyhole` and paste `https://cryptokeyhole.com/api/v1/mcp` as the URL. No OAuth settings are needed.

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

`~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "cryptokeyhole": {
      "httpUrl": "https://cryptokeyhole.com/api/v1/mcp"
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

## Registry entry

[`server.json`](server.json) mirrors the server's entry in the official MCP Registry:

```bash
curl -s 'https://registry.modelcontextprotocol.io/v0.1/servers?search=com.cryptokeyhole'
```

## Licence

The contents of this repository (documentation and configuration examples) are under the [MIT licence](LICENSE). Data served by the MCP server is covered by CryptoKeyhole's [terms of service](https://cryptokeyhole.com/terms/).

Questions or problems: open an [issue](https://github.com/cryptokeyhole/cryptokeyhole-mcp/issues).
