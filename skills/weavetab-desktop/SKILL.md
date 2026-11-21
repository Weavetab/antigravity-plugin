---
name: weavetab-desktop
description: Discover and control live Electron and Chromium-based desktop applications (Discord, Slack, VS Code, Figma, Obsidian, Notion, Postman, Spotify, and local apps) over CDP using Weavetab MCP. Use when automating or inspecting a desktop app instead of a web browser.
---

# Weavetab Desktop App Automation

Weavetab MCP drives Electron and webview desktop apps over CDP using the same toolset as browser tabs. Read the matching reference guide as needed:

- **Discover running Electron apps and debug ports**: Read [references/electron-discovery.md](references/electron-discovery.md) (`browser_detect`).
- **Route tools to desktop apps, multi-window focus, sticky context**: Read [references/desktop-targeting.md](references/desktop-targeting.md) (`app: "<name>"`, `browser_tabs`).

## Core Rules

1. Call `browser_detect` first to check app status (`connected`, `running_no_debug`, `offline`).
2. Always pass `"app": "<app-name>"` on `browser_tabs` and interaction calls targeting a desktop app so Weavetab never opens an unintended browser window.
3. Pass `"app": "browser"` when you explicitly want to target the web browser instead of a configured desktop app.
