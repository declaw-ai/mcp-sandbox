# Brave Search MCP Server (sandboxed)

Web search via the Brave Search API — lets Claude search the internet.

## Setup

1. Get a Brave Search API key from https://brave.com/search/api/
2. Copy `claude-desktop.json` into your Claude Desktop config
3. Replace `YOUR_API_KEY` with your Brave API key
4. Restart Claude Desktop

## Network

This server needs access to:
- `registry.npmjs.org` — npm package install
- `api.search.brave.com` — Brave Search API

Your `BRAVE_API_KEY` can only reach these hosts — even if a dependency is compromised, the key can't be exfiltrated elsewhere.
