# Security Policy

## Supported versions

Heimdall MCP is a hosted service at `https://api.heimdallgov.com/mcp/sse`. The active production version is always the supported one; there are no backported releases.

This repository contains integration docs and configuration examples — security issues in this repo's code (YAML, JSON configs, READMEs) are low-severity and can be filed as normal issues.

## Reporting a vulnerability

**Do not open a public issue for security vulnerabilities.**

Email: **security@heimdallgov.com**

Include:
- A description of the vulnerability and its impact
- Steps to reproduce (include API key scope/tier if auth-related)
- Any PoC code or requests (redact your API key)
- Your contact info for follow-up

We will:
- Acknowledge receipt within **2 business days**
- Provide an initial assessment within **5 business days**
- Keep you informed through remediation
- Credit you in release notes (if you want) once the issue is fixed

## Scope

In scope:
- Authentication/authorization bypass on the MCP gateway (`/mcp/*`)
- Data leakage between tenants (one customer's query returning another's data)
- SSRF, SQL injection, or other server-side vulns in tool implementations
- Rate-limit bypass
- Anything affecting the confidentiality, integrity, or availability of customer data

Out of scope:
- Denial-of-service via high request volume (use the rate-limit error, please)
- Issues in the open-source MCP protocol itself (report upstream to [modelcontextprotocol](https://github.com/modelcontextprotocol))
- Missing security headers on marketing pages
- Social engineering

## Coordinated disclosure

We follow a 90-day coordinated disclosure window. Critical issues may be disclosed sooner after remediation; low-severity issues may be held longer at reporter request.
