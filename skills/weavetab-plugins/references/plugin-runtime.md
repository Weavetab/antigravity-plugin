---
id: plugin-runtime
domain: plugins
triggers:
  - "what plugins are installed"
  - "weavetab plugin"
  - "weavetab plugin use"
  - "weavetab plugin update"
  - "extend mcp"
  - "plugin tools"
  - "plugin commands"
  - "plugin prompts"
  - "plugin resources"
  - "plugin permissions"
tools: [browser_snapshot]
weavetab: ">=2.6.4"
---

# Plugin Runtime: Weavetab Plugin Architecture

Weavetab features an extensible plugin architecture powered by `@weavetab/sdk` (`v2.6.4`, locked 1:1 with `@weavetab/mcp`). Installed plugins can expose:
1. **MCP Tools (`tools`)**: callable directly as MCP tools alongside core tools like `browser_snapshot`.
2. **CLI Commands (`commands`)**: callable from the terminal via `weavetab plugin use <pkg> <command> [--flags]`.
3. **Prompts (`prompts`) & Resources (`resources`)**: `/` Prompts (reusable prompt templates) and `@` Resources (live context URIs).
4. **Agent Instructions (`instructions`)**: a strictly bounded (`<= 80` lines, `<= 3,200` chars) field in `weavetab.json` teaching AI agents how to use the plugin's tools, CLI commands, prompts, and resources without bloating context.

---

## 1. Discovering, Managing & Running Plugins (MCP & CLI)

Connected AI agents can inspect active plugins, their tools, CLI commands, prompts, resources, and operational instructions at any time. When coordinating custom plugin actions with page state, use `browser_snapshot` to capture or verify DOM state before and after plugin execution.

- **Built-In MCP Resources**:
  - `weavetab://instructions`: aggregated bounded operational instructions for all active plugins.
  - `weavetab://commands`: JSON index of all active plugin CLI commands.
  - `weavetab://active-tab`: URL, title, and target ID of the active attached tab.
- **MCP Plugin Manager CLI (`weavetab plugin <subcommand>`)**:
  ```bash
  weavetab plugin add <pkg|./local-path>      # Install from npm or local path (--official, --community, --yes)
  weavetab plugin remove <pkg>                # Uninstall a plugin
  weavetab plugin list                        # List installed plugins, status, commands, and tools
  weavetab plugin info <pkg>                  # Inspect manifest, tools, commands, and permissions
  weavetab plugin enable <pkg>                # Enable a disabled plugin (hot-loads into active MCP sessions)
  weavetab plugin disable <pkg>               # Disable a plugin without uninstalling
  weavetab plugin use <pkg> [cmd] [--flags]   # Run a plugin CLI command, MCP tool, or print instructions
  weavetab plugin update <pkg>                # Upgrade an installed plugin to a newer published version
  weavetab plugin instructions [pkg]          # Print operational instructions & CLI commands
  weavetab plugin browse                      # Interactive terminal explorer to browse & install
  weavetab plugin search [query]              # Search plugins across official registry & npm
  weavetab plugin verify <pkg>                # Check trust tier, registry status, and instructions compliance
  ```

---

## 2. Manifest-Only Permission Model (`weavetab.json`)

Permissions are **manifest-only**: every capability must be declared statically in `weavetab.json`. The MCP loader ignores JS-exported permission objects so every tool call, hook, and proxy call is gated against the static manifest at load time.

| Permission | What it gates |
|---|---|
| `tools.add: true` | Register new MCP tools (`defineTool`) |
| `tools.override: ["browser_navigate"]` | Wrap/replace listed built-in tools (`ctx.mcp.overrideTool`) |
| `tools.remove: ["browser_pdf"]` | Disable listed built-in tools (`ctx.mcp.removeTool`) |
| `commands.add: true` | Register CLI commands (`defineCommand` / `weavetab plugin use`) |
| `prompts.add: true` | Register `/` Prompts (`definePrompt`) |
| `resources.add: true` | Register `@` Resources (`defineResource`) |
| `hooks.onBeforeAction: true` | Intercept tool calls before execution |
| `hooks.onAfterAction: true` | Inspect or transform tool results |
| `hooks.onDOMMutation: true` | Observe real-time DOM mutation / action deltas |
| `hooks.onPageNavigate: true` | React when the active page navigates |
| `extension.hud: true` | Send HUD state and plain-text panels via `PluginExtensionBridge` |
| `extension.overlay: true` | Push viewport overlays and plain-text panels |
| `extension.styles: true` | Inject temporary CSS via `injectStyles` / `injectCss` |
| `extension.scripts: true` | Evaluate scripts (`injectScript`) or render raw HTML in `mountPanel` |
| `desktop.apps: ["discord"]` | Target listed Electron desktop apps via `DesktopProxy` |
| `storage` / `config` / `network` / `cdp` | Sandboxed storage, plugin config, network access, and CDP session access |
| `filesystem.read` / `filesystem.write` | Scoped path arrays for local file access |

Plugins attempting to register capabilities without matching permissions in `weavetab.json` are blocked by both `@weavetab/mcp` at runtime and `@weavetab/sdk` during `weavetab-sdk build`, `weavetab-sdk preflight`, and `weavetab-sdk publish`.

---

## 3. Runtime Host Proxies & Sandboxed Storage

Plugins interact with the browser, desktop apps, and local storage through four typed SDK boundaries:
- **`TabProxy`**: `isAvailable()`, `url()`, `title()`, `evaluate(fn, ...args)`, `waitForSelector(selector, opts)`, `extract(selector, mode)`.
- **`DesktopProxy`**: `isAvailable()`, `listApps()`, `getApp(name)`, `getTargets(portOrName)` (resolves either a numeric CDP port or an app name/alias).
- **`PluginStorage`**: isolated to `~/.weavetab/plugins/<name>/` (`config.json`, `storage.json`, `data/`). Supports `get`, `set`, `delete`, `getJSON`, `setJSON`, `list`, `setWithExpiry`, `getWithExpiry`, `remember`, `clear`, and `pruneExpired()`.
- **`PluginExtensionBridge`**: `send`, `setHudState`, `showThought`, `clearThought`, `injectStyles`/`injectCss`, `injectScript`, `mountPanel`, `clearPanel`, `hideOverlay`, `restoreOverlay`.

---

## 4. Developer SDK CLI (`weavetab-sdk`)

When `@weavetab/mcp` is installed globally, use `weavetab-sdk` (or `npx @weavetab/sdk`) for plugin authoring and `weavetab plugin` for runtime management:

```bash
weavetab-sdk init [name] [--template <id>]  # Scaffold plugin (standard, visual, scraper, automation, override)
weavetab-sdk build                          # Compile TypeScript, verify instructions budget, sync weavetab.json
weavetab-sdk dev                            # Watch mode (tsc --watch) with automatic weavetab.json sync
weavetab-sdk validate [--offline]           # Validate weavetab.json against schema
weavetab-sdk preflight [dir]                # Audit manifest, permissions, entry path containment & budget
weavetab-sdk publish [--dry-run]            # Run preflight gate, npm publish, and weavetab.dev registry sync
npm run release                             # Scaffolded package script that runs weavetab-sdk publish
```

Additional helper commands: `weavetab-sdk templates`, `weavetab-sdk manifest` (`schema`), `weavetab-sdk doctor`, and `weavetab-sdk version`.

---

## 5. Strict `instructions` Budget Enforcement

To prevent third-party plugins from overflowing an AI agent's context window, `@weavetab/sdk` enforces hard limits on `instructions` in `weavetab.json` during `weavetab-sdk build`, `weavetab-sdk validate`, `weavetab-sdk preflight`, `weavetab-sdk doctor`, and `weavetab-sdk publish`:
- **Maximum Lines**: `80` lines
- **Maximum Characters**: `3,200` characters (`~800` tokens)

Any plugin exceeding these limits is rejected before it can be built or published, and the MCP runtime also clamps `instructions` at load time as defense-in-depth.

---

## 6. Trust Tiers

| Tier | Badge | Condition |
|---|---|---|
| Official | `✚ [Official]` | `@weavetab/plugin-*` packages |
| Verified | `✔ [Verified]` | Listed in the Weavetab verified registry |
| Unverified | `⚠ [Unverified]` | Unknown community package |
