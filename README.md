# Heimdall MCP Server

**Federal oversight intelligence for AI agents.** Heimdall gives Claude, Cline, Cursor, and any MCP-compatible agent direct access to 197K+ pages of federal oversight data — GAO reports, Inspector General findings, Single Audit (FAC) results, SAM.gov opportunities, and contractor risk profiles — via 23 purpose-built tools.

Ask your agent:

> *"Find open GAO recommendations about DOT cybersecurity from the last 18 months."*

> *"Show me OIG findings against our prospect before the kickoff call."*

> *"Which NAICS codes are growing in federal spending this quarter?"*

## Quickstart

Heimdall MCP is a **hosted** server — nothing to install or run locally. Point your agent at the public SSE endpoint and authenticate with an API key.

1. **Get an API key** — Sign up at [heimdallgov.com](https://heimdallgov.com). Free trial available; the $9/mo Core plan includes MCP access.
2. **Configure your client** (pick one below).
3. **Restart the client** and start asking oversight-data questions.

### Claude Desktop

Edit `%APPDATA%/Claude/claude_desktop_config.json` (Windows) or `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS):

```json
{
  "mcpServers": {
    "heimdall": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://api.heimdallgov.com/mcp/sse",
        "--header",
        "Authorization: Bearer YOUR_HEIMDALL_API_KEY"
      ]
    }
  }
}
```

### Cline (VS Code)

Open the Cline MCP settings panel and add:

```json
{
  "mcpServers": {
    "heimdall": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://api.heimdallgov.com/mcp/sse",
        "--header",
        "Authorization: Bearer YOUR_HEIMDALL_API_KEY"
      ]
    }
  }
}
```

### Cursor

Add to `.cursor/mcp.json` in your project (or global settings):

```json
{
  "mcpServers": {
    "heimdall": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://api.heimdallgov.com/mcp/sse",
        "--header",
        "Authorization: Bearer YOUR_HEIMDALL_API_KEY"
      ]
    }
  }
}
```

> Full config examples for each client are in [`configs/`](configs/).

## What you get

**36 tools** across nine domains:

| Domain | Tools | What it covers |
|---|---|---|
| **Contracts** | 4 | Search 58.5M FPDS actions; recompete/expiring-contract lookups; vendor award history; full PIID modification history |
| **Contractor Risk & Performance** | 2 | 6-dim composite risk score + 5-subdim performance score across 872,793 entities (Heimdall differentiator) |
| **Grants** | 3 | Open opportunities (Grants.gov); historical awards (USAspending); Single Audit findings (FAC) |
| **Legislation** | 2 | Congressional bill search + detail |
| **Appropriations** | 1 | Agency/bureau/account budget with 3-year trend |
| **Findings & Research** | 12 | GAO/IG/FAC findings; source documents; entity extraction; cross-agency patterns; LDA topics; TF-IDF similarity; findings-by-contractor; RAG over 197K PDF chunks |
| **Sentinel** (live oversight monitoring) | 6 | 70+ OIG sites scraped nightly; change detection; anomaly alerts; scraper health |
| **Pipeline & Market Intel** | 6 | Ranked opportunities by profile; NAICS spending trends; cost tracking; AI usage logs |
| **Write ops** | gated, auditable | `acknowledge_alert`, `approve_rule_proposal` |

**11 resources** + **7 prompts** for structured workflows. See [`docs/tools.md`](docs/tools.md) for the full tool reference with parameters and examples.

## Who it's for

- **CPA firms & compliance consultants** — subcontracting scope, OIG risk on clients, FAC audit flags
- **Federal contractors** — competitor GAO findings, recompete signals, NAICS market sizing
- **Oversight researchers & journalists** — cross-agency pattern detection, source PDFs with citations
- **Risk & audit teams** — priority recommendations by agency, cost exposure, historical baselines

## Pricing

| Plan | $/mo | MCP access | Use case |
|---|---|---|---|
| **Free Trial** | $0 | REST only | Evaluate |
| **Core** | $9 | ✅ 23 tools | Individual analysts, small teams |
| **API** | $49 | ✅ 23 tools + higher rate limits | Automation, integrations |
| **Pro** | $250 ($2,999/yr) | ✅ everything + custom profiles, Opus chat | Firms, power users |
| **Enterprise** | custom | ✅ + SSO, SLA, dedicated support | Agencies, prime contractors |

Sign up at [heimdallgov.com](https://heimdallgov.com). Billing via Stripe.

## Security & data

- **Auth:** Bearer API key (`hml_sk_...`) over TLS 1.2+. Keys rotate at will via the dashboard.
- **Data scope:** public federal oversight data only (GAO.gov, OIG sites, FAC, SAM.gov, Federal Register). No PII, no classified content.
- **Write ops:** only two tools mutate state (`acknowledge_alert`, `approve_rule_proposal`) and only on your own subscription's data.
- **Rate limits:** per-tier, enforced at the API gateway. See pricing page for details.
- **SOC 2:** in progress (target 2026-Q4). For now, security questions to [security@heimdallgov.com](mailto:security@heimdallgov.com).

Report vulnerabilities: see [SECURITY.md](SECURITY.md).

## Troubleshooting

**`HTTP 401 unauthorized`** — missing or malformed `Authorization: Bearer` header.

**`HTTP 403 Active subscription required`** — your key is on the free/trial tier. Upgrade to Core ($9/mo) at [heimdallgov.com/pricing](https://heimdallgov.com/pricing).

**`HTTP 429 rate limit exceeded`** — slow down, or upgrade tier for higher limits.

**Claude Desktop won't load the server** — check `%APPDATA%/Claude/logs/mcp*.log`. Usually a JSON syntax error in the config or an outdated `mcp-remote`. Try `npx -y mcp-remote@latest`.

**SSE connection drops after ~60s** — report to [support@heimdallgov.com](mailto:support@heimdallgov.com) with a timestamp; probable CloudFront idle-timeout tuning on our side.

## Links

- 🌐 Website: [heimdallgov.com](https://heimdallgov.com)
- 📚 Tool reference: [`docs/tools.md`](docs/tools.md)
- 🐛 Issues: [github.com/getgovernmentmoney/heimdall-mcp/issues](https://github.com/getgovernmentmoney/heimdall-mcp/issues)
- ✉️ Support: [support@heimdallgov.com](mailto:support@heimdallgov.com)
- 🐦 Updates: [@heimdallgov](https://x.com/heimdallgov)

## License

MIT — see [LICENSE](LICENSE). The MCP server runs on Heimdall's infrastructure; this repository contains integration docs, configuration examples, and the `server.json` manifest. The server's source code is proprietary.
