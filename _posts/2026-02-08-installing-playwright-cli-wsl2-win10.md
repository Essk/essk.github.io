---
title: Installing playwright-cli in WSL2/Win10
date: 2026-02-08
layout: default
---

## Overview

Got playwright-cli working in headless mode on WSL2 (Windows 10). Required some dependency wrestling.

## Installation Steps

### 1. Install Chromium browser

```bash
cd /home/sarah/.claude/skills/playwright-cli
npx playwright install chromium
```

Downloads ~280 MB of browser binaries to `~/.cache/ms-playwright/`.

### 2. Install system dependencies

This is the critical step. Chromium needs various shared libraries that aren't included:

```bash
sudo env "PATH=$PATH" npx playwright install-deps chromium
```

**Note the PATH preservation** - `sudo` doesn't inherit your user PATH by default, so `npx` won't be found without this workaround.

Installs packages like `libnspr4`, `libnss3`, and other dependencies.

### 3. Run playwright-cli

```bash
playwright-cli open http://localhost:3000/ur/test/app --browser=chromium
```

## What Works

- **Headless mode** works perfectly
- **Snapshots** provide structured YAML view of page elements with reference IDs
- **Element interaction** via refs (e.g., `playwright-cli click e15`)
- **Screenshots, PDFs, JavaScript evaluation**
- **Browser sessions** with persistent or in-memory profiles

## Headed Mode (Not Tested)

To run headed browsers in WSL2 on Windows 10:

1. Install X server on Windows (VcXsrv, Xming, X410)
2. Configure X server to accept connections
3. Set DISPLAY variable in WSL2: `export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):0`
4. May need Windows Firewall rules for port 6000

**Windows 11** has WSLg built-in and doesn't need this setup.

## Alternative: Chrome DevTools MCP

The Chrome DevTools MCP server can also control browsers via DevTools Protocol. Already had a connection available during testing - might be easier than playwright-cli for some use cases.

## Key Takeaway

Headless browser automation in WSL2 is straightforward once dependencies are installed. The `sudo env "PATH=$PATH"` trick is the key to getting system packages installed correctly.
