# mcp-sandbox

Sandbox any MCP server in one line. Firecracker microVM isolation for Claude Desktop, Cursor, Windsurf, Claude Code, and every MCP client.

[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![Declaw CLI](https://img.shields.io/github/v/release/declaw-ai/declaw-cli?label=declaw%20cli)](https://github.com/declaw-ai/declaw-cli/releases)

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

Your GitHub token is accessible to the MCP server *and* its entire dependency tree — 847 transitive npm packages running with full host access.

**After** — sandboxed in a Firecracker microVM:

```json
{
  "mcpServers": {
    "github": {
      "command": "declaw",
      "args": ["mcp", "--env", "GITHUB_PERSONAL_ACCESS_TOKEN", "--network-allow", "registry.npmjs.org,api.github.com,github.com,codeload.github.com", "--", "npx", "-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_..." }
    }
  }
}
```

Same MCP server. Same functionality. But now your token can only reach GitHub — even if a dependency is compromised, it can't exfiltrate credentials anywhere else. Only the env vars you explicitly forward with `--env` reach the sandbox.

## Why

MCP servers that connect to external APIs handle your most sensitive credentials — GitHub tokens, Slack bot tokens, API keys, database credentials. These servers run as subprocesses with full host access: your files, your SSH keys, your network.

This isn't theoretical:
- Claude Desktop Extensions had a [zero-click RCE](https://layerxsecurity.com/blog/claude-desktop-extensions-rce/) rated CVSS 10/10 (LayerX, Feb 2026)
- Cursor had [CVE-2025-54135](https://www.tenable.com/cve/CVE-2025-54135) (CurXecute, CVSS 9.8) and [CVE-2025-54136](https://www.tenable.com/cve/CVE-2025-54136) (MCPoison, CVSS 8.8)

`declaw mcp` wraps any stdio MCP server in a Firecracker microVM with network deny-all by default. The server works identically — it just can't reach anything you didn't explicitly allow.

## When to use this

`declaw mcp` is designed for **MCP servers that talk to external APIs with credentials**:

| Server | Credentials at risk | Why sandbox it |
|--------|-------------------|----------------|
| GitHub | `GITHUB_PERSONAL_ACCESS_TOKEN` | Token can only reach `api.github.com`, not exfiltrated elsewhere |
| Slack | `SLACK_BOT_TOKEN` | Bot token confined to `api.slack.com` |
| Brave Search | `BRAVE_API_KEY` | API key confined to `api.search.brave.com` |
| Database | `DATABASE_URL` | Connection string can't be sent to external hosts |
| Any API server | API keys, tokens, secrets | Network allowlist = credential containment |

**Not a fit for:** MCP servers that need local host access (filesystem, SQLite, etc.) — these need your local files to be useful, which a cloud sandbox intentionally prevents.

## Install

```bash
# macOS / Linux
curl -fsSL https://raw.githubusercontent.com/declaw-ai/declaw-cli/main/install.sh | sh

# or with Go
go install github.com/declaw-ai/declaw-cli/cmd/declaw@latest

# or download binary
# https://github.com/declaw-ai/declaw-cli/releases
```

Then sign up and authenticate:

```bash
# 1. Create a free account at https://console.declaw.ai
# 2. Copy your API key from the dashboard
# 3. Authenticate:
declaw auth login
```

## Client Setup

### Claude Desktop

Config path: `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows)

```json
{
  "mcpServers": {
    "github": {
      "command": "declaw",
      "args": ["mcp", "--env", "GITHUB_PERSONAL_ACCESS_TOKEN", "--network-allow", "registry.npmjs.org,api.github.com,github.com,codeload.github.com", "--", "npx", "-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_..." }
    }
  }
}
```

### Cursor

Config path: `~/.cursor/mcp.json` — same JSON structure as above.

### Windsurf

Config path: `~/.codeium/windsurf/mcp_config.json` — same JSON structure as above.

### Claude Code

```bash
claude mcp add github -e GITHUB_PERSONAL_ACCESS_TOKEN=ghp_... -- declaw mcp --env GITHUB_PERSONAL_ACCESS_TOKEN --network-allow registry.npmjs.org,api.github.com,github.com,codeload.github.com -- npx -y @modelcontextprotocol/server-github
```

## Examples

See [`examples/`](examples/) for ready-to-use configs:

- [`github`](examples/github/) — GitHub API (repos, issues, PRs, code search)
- [`brave-search`](examples/brave-search/) — Web search via Brave Search API
- [`fetch`](examples/fetch/) — Web content fetching and conversion

## How it works

`declaw mcp` is a transparent stdio forwarder. It creates a Firecracker microVM, starts the MCP server inside it, and forwards JSON-RPC messages between the MCP client and the sandboxed server. The client doesn't know anything changed. The server doesn't know it's sandboxed.

Network is deny-all by default. Use `--network-allow` to open specific hosts the server needs. This is the key security property: credentials passed to the server can only reach hosts you explicitly permit.

## Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--network-allow <hosts>` | deny-all | Comma-separated outbound hostname allowlist |
| `--template <name>` | `mcp-server` | Sandbox template (default includes Node.js + Python) |
| `--timeout <seconds>` | `86400` | Sandbox timeout (default 24h) |
| `--env KEY` or `--env KEY=VAL` | — | Environment variable to forward (repeatable). `KEY` reads from host env; `KEY=VAL` sets explicitly. |
| `--verbose` | off | Diagnostic logging to stderr |

## Custom dependencies

The default `mcp-server` template includes Node.js and Python, which covers most MCP servers. If your server needs additional system packages (e.g., `ffmpeg`, native libraries), build a custom template:

```bash
# Create a Dockerfile
echo 'FROM declaw/mcp-server:latest
RUN apt-get update && apt-get install -y ffmpeg' > Dockerfile

# Build it (returns a template ID)
declaw template build --dockerfile Dockerfile

# Use the template ID from the build output
declaw mcp --template <template-id> -- your-server-command
```

See `declaw template build --help` for details.

## Links

- [Full CLI documentation](https://github.com/declaw-ai/declaw-cli)
- [Declaw docs](https://docs.declaw.ai)
- [declaw.ai](https://declaw.ai)

## License

Apache 2.0
