# PlayTesterHub MCP server

Let your AI agent (Claude, ChatGPT, Cursor, Claude Code…) order and follow a Google Play closed test on 12 real Android phones.

Address: `https://playtesterhub.com/api/mcp` ([details](playtesterhub/)). Hosted remote MCP server (Streamable HTTP) with OAuth or an API key: nothing to install. Your agent never pays on your behalf: orders return a payment link that a human opens.

## Quick start

**Claude.ai / Claude Desktop:** Settings › Connectors › Add custom connector, paste the address, then sign in and click Allow.

**ChatGPT:** Settings › Apps & Connectors › Advanced settings (developer mode, depending on plan) › create an app with the address and OAuth.

**Claude Code:**

```bash
claude mcp add --transport http playtesterhub https://playtesterhub.com/api/mcp
```

**Cursor** (`~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "playtesterhub": { "url": "https://playtesterhub.com/api/mcp" }
  }
}
```

Labels in each app may vary with its version.

## Links

- PlayTesterHub API and MCP: https://playtesterhub.com/en/api-mcp
- Free guide to Google Play closed testing: https://github.com/Hangarmobile/google-play-closed-testing

This repository contains configuration and documentation only. Licensed under MIT.
