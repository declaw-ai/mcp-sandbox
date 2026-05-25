# Fetch MCP Server (sandboxed)

Fetches web pages and converts them to markdown for Claude to read.

## Setup

1. Copy `claude-desktop.json` into your Claude Desktop config
2. Restart Claude Desktop

No API key required.

## Network

This server fetches arbitrary URLs, so `--network-allow` includes `*` (all outbound).
The `pypi.org` and `files.pythonhosted.org` entries are needed for the initial `pip install`.

If you want to restrict fetching to specific domains, replace `*` with a comma-separated list (e.g., `pypi.org,files.pythonhosted.org,docs.python.org,en.wikipedia.org`).

## Template

Uses `--template python` (provides Python 3 + pip).
