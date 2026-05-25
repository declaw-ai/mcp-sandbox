# GitHub MCP Server (sandboxed)

Gives Claude access to GitHub — search repos, read code, create issues, open PRs.

## Setup

1. Get a GitHub Personal Access Token from https://github.com/settings/tokens
2. Copy `claude-desktop.json` into your Claude Desktop config
3. Replace `YOUR_TOKEN_HERE` with your token
4. Restart Claude Desktop

## Network

This server needs access to:
- `api.github.com` — GitHub API
- `github.com` — GitHub web
- `codeload.github.com` — code downloads

## Template

Uses `--template node` (provides Node.js + npx).
