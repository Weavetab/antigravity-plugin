# Weavetab Operational Rules

## Core Browser & Electron Rules
- Call `browser_detect` at the start of a session before assuming a browser or Electron desktop app is attached.
- Prefer `browser_find` for targeted lookups and `browser_map` with `lite: true` (plus `delta: true` on follow-ups) instead of full DOM dumps.
- Act on volatile `w:NN` refs returned by Weavetab tools rather than inventing brittle CSS selectors.
- Pass all credentials, passwords, 2FA seeds, and sensitive values through `browser_type_secret`, never `browser_type` or `browser_fill`.
- Pass `app: "<name>"` explicitly on `browser_tabs` and interaction calls when targeting an Electron desktop app; pass `app: "browser"` when targeting Chrome explicitly.
- Never bypass CAPTCHAs or anti-bot gates automatically; pause and ask the user to complete them in their live window.

## Weavetab Plugins & CLI Execution
- **Plugins are not automatically injected into your context** like the 40 built-in MCP tools. Installed plugins can expose CLI commands (`commands`), utilities (`prompts` / `resources`), and plugin-scoped tools on top of the core server.
- **Discover installed plugins & instructions**: Check the `weavetab://plugins` and `weavetab://plugins/<slug>/instructions` MCP resources, or run `weavetab plugin list`, `weavetab plugin info <pkg>`, and `weavetab plugin instructions <pkg>` in the terminal.
- **Run plugin commands and tools via CLI**:
  - `weavetab plugin use <pkg_or_alias> <command_or_tool> [--flags]`
  - Shorthand: `weavetab use <pkg_or_alias> <command_or_tool> [--flags]`
- **Manage plugins**: `weavetab plugin add|remove|update|enable|disable|search|browse|verify <pkg>`.
- **Security**: Plugin permissions are strictly manifest-only in `weavetab.json`.
