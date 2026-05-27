# mcp-sandbox

Sandbox any MCP server in one line. Firecracker microVM isolation for Claude Desktop, Cursor, Windsurf, Claude Code, and every MCP client.

[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![GitHub release](https://img.shields.io/github/v/release/declaw-ai/declaw-cli)](https://github.com/declaw-ai/declaw-cli/releases)

## Before / After

**Before** — no sandbox:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_..." }
    }
  }
}
```

**After** — sandboxed in a Firecracker microVM:

```json
{
  "mcpServers": {
    "github": {
      "command": "declaw",
      "args": ["mcp", "--template", "node", "--network-allow", "api.github.com,github.com,codeload.github.com", "--", "npx", "-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_..." }
    }
  }
}
```

One prefix. No code changes. The MCP server runs unchanged inside a hardware-isolated sandbox.

## Why

MCP servers run as subprocesses with full host access — your files, your SSH keys, your credentials, your network. Claude Desktop Extensions had a [zero-click RCE](https://layerxsecurity.com/blog/claude-desktop-extensions-rce/) rated CVSS 10/10 (LayerX, Feb 2026). Cursor had [CVE-2025-54135](https://www.tenable.com/cve/CVE-2025-54135) and [CVE-2025-54136](https://www.tenable.com/cve/CVE-2025-54136). `declaw mcp` wraps any stdio MCP server in a Firecracker microVM. The server runs unchanged — it just can't touch your machine.

## Install

```bash
# macOS / Linux
curl -fsSL https://raw.githubusercontent.com/declaw-ai/declaw-cli/main/install.sh | sh

# or with Go
go install github.com/declaw-ai/declaw-cli/cmd/declaw@latest

# or download binary
# https://github.com/declaw-ai/declaw-cli/releases
```

Then authenticate:

```bash
declaw auth login
# or: export DECLAW_API_KEY=your-key
```

## Client Setup

### Claude Desktop

Config path: `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows)

```json
{
  "mcpServers": {
    "github": {
      "command": "declaw",
      "args": ["mcp", "--template", "node", "--network-allow", "api.github.com,github.com,codeload.github.com", "--", "npx", "-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_..." }
    }
  }
}
```

### Cursor

Config path: `~/.cursor/mcp.json` — same JSON structure as above.

### Claude Code

```bash
claude mcp add github -- declaw mcp --template node --network-allow api.github.com,github.com,codeload.github.com -- npx -y @modelcontextprotocol/server-github
```

## Examples

See [`examples/`](examples/) for ready-to-use configs for popular MCP servers:

- [`github`](examples/github/) — GitHub API access (repos, issues, PRs, code search)
- [`brave-search`](examples/brave-search/) — Web search via Brave Search API
- [`fetch`](examples/fetch/) — Web content fetching and conversion
- [`filesystem`](examples/filesystem/) — Sandboxed file read/write
- [`memory`](examples/memory/) — Knowledge graph memory (persistent across conversations)

## How it works

`declaw mcp` is a transparent stdio forwarder. It creates a Firecracker microVM, starts the MCP server inside it, and forwards JSON-RPC messages between the MCP client and the sandboxed server. The client doesn't know anything changed. The server doesn't know it's sandboxed.

Network is deny-all by default. Use `--network-allow` to open specific hosts the server needs.

## Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--network-allow <hosts>` | deny-all | Comma-separated outbound hostname allowlist |
| `--template <name>` | `base` | Sandbox template (`node` for npx servers, `python` for pip servers) |
| `--timeout <seconds>` | `86400` | Sandbox timeout (default 24h) |
| `--env KEY=VAL` | — | Environment variable (repeatable) |
| `--verbose` | off | Diagnostic logging to stderr |

## Links

- [Full CLI documentation](https://github.com/declaw-ai/declaw-cli)
- [Declaw docs](https://docs.declaw.ai)
- [declaw.ai](https://declaw.ai)

## License

Apache 2.0
