# Memory MCP Server (sandboxed)

A knowledge graph memory server — lets Claude store and retrieve structured information across conversations.

## Setup

1. Copy `claude-desktop.json` into your Claude Desktop config
2. Restart Claude Desktop

No API key required.

## Network

This server only needs network access for the initial `npx` install:
- `registry.npmjs.org` — npm package install

After install, the server runs entirely offline inside the sandbox.

## Template

Uses `--template node` (provides Node.js + npx).
