# Filesystem MCP Server (sandboxed)

Gives Claude read/write access to the filesystem — safely isolated inside the sandbox.

## Setup

1. Copy `claude-desktop.json` into your Claude Desktop config
2. Restart Claude Desktop

No API key required.

## Network

This server only needs network access for the initial `npx` install:
- `registry.npmjs.org` — npm package install

After install, the server runs entirely offline inside the sandbox.

## What gets sandboxed

The filesystem server runs inside a Firecracker microVM. It can only see `/home/user` inside the sandbox — it has no access to your host machine's files. Use `declaw sandbox files` to upload/download files to the sandbox if needed.

## Template

Uses `--template node` (provides Node.js + npx).
