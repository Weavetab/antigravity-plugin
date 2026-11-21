---
name: weavetab-browser
description: Drive live web browsers over direct CDP with Weavetab MCP. Covers 4-tier token-efficient DOM reading (browser_find, browser_map lite/delta, browser_scrape, visual grounding), clicking/typing/pointer pipelines, multi-field form filling, and tab/window management. Use for any browser inspection or interaction task via Weavetab.
---

# Weavetab Browser Automation

Use Weavetab MCP (`@weavetab/mcp@latest`) for low-token browser control over local CDP. Read only the reference guide matching the immediate step:

- **Inspect DOM, locate elements, extract text, conserve tokens**: Read [references/dom-strategy.md](references/dom-strategy.md) (`browser_find`, `browser_map`, `browser_scrape`, `browser_snapshot`).
- **Click, type, keys, pointer trajectories, uploads**: Read [references/interaction-pipeline.md](references/interaction-pipeline.md) (`browser_click`, `browser_type`, `browser_key`, `browser_pointer`, `browser_upload`).
- **Fill multi-field forms, dropdowns, checkboxes**: Read [references/form-automation.md](references/form-automation.md) (`browser_fill`, `browser_select`).
- **Manage tabs, windows, and tab crash recovery**: Read [references/tab-and-window.md](references/tab-and-window.md) (`browser_tabs`, `browser_navigate`).

## Core Rules

1. Call `browser_detect` first to check live browser attachment before running actions.
2. Prefer `browser_find` for known labels/placeholders, and always pass `lite: true` (plus `delta: true` on follow-ups) to `browser_map`.
3. Act on volatile `w:NN` refs returned by Weavetab instead of inventing CSS selectors.
