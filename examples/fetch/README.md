# Fetch MCP Server (sandboxed)

Fetches web pages and converts them to markdown for Claude to read.

## Setup

1. Copy `claude-desktop.json` into your Claude Desktop config
2. Restart Claude Desktop

No API key required.

## Network

This server fetches arbitrary URLs, so `--network-allow` uses `*` (all outbound). This also covers the initial `pip install` from PyPI.

To restrict fetching to specific domains, replace `*` with a comma-separated list (e.g., `pypi.org,files.pythonhosted.org,docs.python.org,en.wikipedia.org`).

Even with `*`, the server is still fully sandboxed — it can't access your host filesystem, SSH keys, or other credentials. The isolation is the VM, the network allowlist is an additional layer.
