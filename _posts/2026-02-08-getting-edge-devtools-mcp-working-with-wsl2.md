---
title: Getting Edge DevTools MCP Working with WSL2
date: 2026-02-08
layout: default
---

## Problem

Needed Claude to interact with Microsoft Edge for debugging video playback issues that only occur in Edge, similar to the existing Chrome DevTools MCP setup.

## Solution

Edge uses the Chrome DevTools Protocol (CDP), so the same `chrome-devtools-mcp` package works for both browsers. Just need separate instances pointing to different ports.

## Steps

### 1. Launch Edge with Remote Debugging

```bash
"C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" --remote-debugging-port=9333 --user-data-dir="C:\temp\edge-debug-profile"
```

I'm using `:9333` here so I can keep running Chrome on `:9222`

### 2. Set Up Port Forwarding (Windows PowerShell as Admin)

```powershell
netsh interface portproxy add v4tov4 listenaddress=XXX.XXX.XXX.XXX listenport=9333 connectaddress=127.0.0.1 connectport=9333
```

Where `XXX.XXX.XXX.XXX` is your WSL2 host IP (get via `ip route` in WSL).

### 3. Test Connection from WSL

```bash
curl http://XXX.XXX.XXX.XXX:9333/json/version
```

Should return Edge version info.

### 4. Update `~/.claude.json`

Add second MCP server config:

```json
"mcpServers": {
  "chrome-devtools": {
    "type": "stdio",
    "command": "npx",
    "args": [
      "chrome-devtools-mcp@latest",
      "--browserUrl",
      "http://XXX.XXX.XXX.XXX:9222"
    ],
    "env": {}
  },
  "edge-devtools": {
    "type": "stdio",
    "command": "npx",
    "args": [
      "chrome-devtools-mcp@latest",
      "--browserUrl",
      "http://XXX.XXX.XXX.XXX:9333"
    ],
    "env": {}
  }
}
```

### 5. Restart Claude Code

After restart, both `mcp__chrome-devtools__*` and `mcp__edge-devtools__*` tools are available.

## Key Findings

- There's a community `edge-devtools-mcp` package on GitHub, but it's not published to npm and lacks proper npx setup
- No need for a separate package — `chrome-devtools-mcp` works perfectly with Edge since they use the same protocol
- Can run both Chrome and Edge with debugging simultaneously on different ports
- Same port forwarding approach works for both browsers

## Use Case

Perfect for debugging browser-specific issues (e.g., video playback problems) where you need Claude to inspect console logs, network requests, and page state directly in the affected browser.
