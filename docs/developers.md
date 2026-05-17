# Developers

Thalian exposes a hosted MCP server, a REST API, and webhook events for teams that want to integrate Thalian data into their own workflows, AI assistants, or tooling.

---

## MCP Server

Connect any MCP-compatible AI assistant — Claude Code, Cursor, Windsurf — directly to your Thalian workspace. Query live findings, identity posture, and device compliance, or trigger syncs, all from your AI client.

- Hosted at `mcp.thalian.ai` — no self-hosting required
- Authenticate with an API key from **Settings → API Keys**
- Available on Pro and Enterprise

[MCP Server setup →](./mcp-server.md)

## REST API

Programmatic access to your workspace data. Use the API to pull findings into your own dashboards, build custom automations, or integrate Thalian data into your SIEM or ticketing system.

- Base URL: `https://app.thalian.ai/api/`
- Bearer token authentication via API keys
- Rate limit: 1,000 requests/hour on Pro

[API Reference →](./api-reference.md)

## Webhooks

Thalian can push events to your endpoint when findings are created or resolved, when remediation actions complete, and when integrations disconnect. Set up webhooks under **Settings → Webhooks**.

[Webhooks setup →](./integrations/webhooks.md)

## API Keys

All developer surfaces — MCP, REST API, and webhooks — use API keys for authentication. Keys are created and managed in **Settings → API Keys**.

- Up to 10 active keys per workspace
- Keys are scoped to the workspace they were created in
- The full key value is shown only once at creation — store it securely
- Keys can be revoked at any time

---

*For general workspace configuration, see [Settings & Admin](./settings-and-admin.md).*
