---
title: Getting Chrome DevTools MCP working with WSL2/Win10
date: 2026-02-07
layout: default
---

## Problem

Chrome DevTools MCP server in WSL2 on Windows 10 couldn't connect to Chrome running on Windows. The default `localhost:9222` configuration doesn't work because WSL2 uses a separate virtual network adapter with inconsistent localhost forwarding on Win10.

## Solution

### 1. Launch Chrome with debugging enabled

```cmd
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --remote-debugging-address=0.0.0.0 --user-data-dir="C:\temp\chrome-debug"
```

**Key flags:**
- `--remote-debugging-port=9222` — enables debugging protocol
- `--remote-debugging-address=0.0.0.0` — binds to all interfaces (not just 127.0.0.1)
- `--user-data-dir="C:\temp\chrome-debug"` — uses clean profile to avoid conflicts with existing Chrome instances

### 2. Set up port proxy from WSL to Windows

Windows Firewall blocks WSL2 → Windows connections by default. Use netsh port proxy in PowerShell as Administrator:

```powershell
netsh interface portproxy add v4tov4 listenaddress=XXX.XXX.XXX.XXX listenport=9222 connectaddress=127.0.0.1 connectport=9222
```

Replace `XXX.XXX.XXX.XXX` with your Windows host IP from WSL (check with `cat /etc/resolv.conf | grep nameserver`).

### 3. Configure MCP server

Update `~/.claude.json`:

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
  }
}
```

Restart Claude Code for the config to take effect.

## Verification

Test from WSL:
```bash
curl http://XXX.XXX.XXX.XXX:9222/json/version
```

Should return Chrome version info as JSON.

## Notes

- Firewall rules alone didn't work — needed the netsh port proxy
- Chrome must be fully closed before launching with debugging flags
- The `--user-data-dir` flag is critical if Chrome is already running with a different profile
- Win11 and newer WSL builds have better localhost forwarding and may not need the port proxy workaround
