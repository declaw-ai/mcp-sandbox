# Fetch MCP Server (sandboxed)

Fetches web pages and converts them to markdown for Claude to read.

## Setup

1. Copy `claude-desktop.json` into your Claude Desktop config
2. Restart Claude Desktop

No API key required.

## Network

This server fetches arbitrary URLs, so `--network-allow` uses `*` (all outbound). This also covers the initial `pip install` from PyPI.

To restrict fetching to specific domains, replace `*` with a comma-separated list (e.g., `pypi.org,files.pythonhosted.org,docs.python.org,en.wikipedia.org`).

## Template

Uses `--template python` (provides Python 3 + pip).
