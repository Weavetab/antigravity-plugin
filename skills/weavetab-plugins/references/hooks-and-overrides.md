---
id: hooks-and-overrides
domain: plugins
triggers:
  - "plugin hooks"
  - "before tool call"
  - "after tool call"
  - "telemetry interception"
  - "sdk hooks"
tools: []
weavetab: ">=2.6.4"
---

# Plugin Hooks & Lifecycle Overrides

Plugins built with `@weavetab/sdk` register lifecycle hooks, tool overrides, and host proxy calls gated strictly by static permissions in `weavetab.json`. The MCP loader ignores JS-exported permission objects so every hook and override is auditable before execution.

---

## 1. Supported Plugin Hooks

The `PluginHooks` interface exposes four interceptors (requiring matching `hooks.*` permissions in `weavetab.json`):

1. **`onBeforeAction(handler)`** (*alias:* `beforeToolCall`, permission: `hooks.onBeforeAction`)
   - `handler(tool, args, session)` runs **before** any tool executes.
   - May return a modified `args` object (replacing the arguments) or `{ __blocked: true, reason }` to abort the call.
   - Typical uses: argument validation, tenant/header injection, domain or action guardrails.
2. **`onAfterAction(handler)`** (*alias:* `afterToolCall`, permission: `hooks.onAfterAction`)
   - `handler(tool, result, session, args)` runs **after** a tool completes.
   - Its return value replaces the tool result for telemetry, timing, or enriched output.
3. **`onDOMMutation(handler)`** (permission: `hooks.onDOMMutation`)
   - `handler(delta, session)` observes real-time DOM mutation and action deltas (`added`, `removed`, `changed`, `significantChange`, `stable_count`).
4. **`onPageNavigate(handler)`** (permission: `hooks.onPageNavigate`)
   - `handler(url, session)` runs when the active page navigates to a new URL.

---

## 2. Built-In Tool Overrides & Removals

Plugins can wrap or disable any of the 40 built-in MCP tools when declared in `weavetab.json`:

- **`ctx.mcp.overrideTool(toolName, handler)`** (requires `permissions.tools.override: ["<toolName>"]`): wraps or replaces a built-in tool; receives `(args, session, config, originalHandler)`.
- **`ctx.mcp.removeTool(toolName)`** (requires `permissions.tools.remove: ["<toolName>"]`): hides a built-in tool from the active MCP surface.

---

## 3. Host Proxies Available in Plugin Contexts

Tool handlers, command handlers, and lifecycle callbacks interact with the host through sandboxed proxies:

- **`TabProxy` (`callCtx.tab`)**: `isAvailable()`, `url()`, `title()`, `evaluate(fn, ...args)`, `waitForSelector(selector, opts)`, `extract(selector, mode)`.
- **`DesktopProxy` (`callCtx.desktop`)**: `isAvailable()`, `listApps()`, `getApp(name)`, `getTargets(portOrName)` (accepts a CDP port number or desktop app name/alias like `"vscode"`).
- **`PluginStorage` (`ctx.storage` / `callCtx.storage`)**: scoped to `~/.weavetab/plugins/<name>/` (`get`, `set`, `delete`, `getJSON`, `setJSON`, `getPath`, `list`, `setWithExpiry`, `getWithExpiry`, `remember`, `clear`, `pruneExpired()`).
- **`PluginExtensionBridge` (`ctx.extension` / `callCtx.extension`)**: `send`, `setHudState`, `showThought`, `clearThought`, `injectStyles`/`injectCss`, `injectScript`, `mountPanel`, `clearPanel`, `hideOverlay`, `restoreOverlay`.

---

## 4. Agent Operational Awareness

- A blocked action surfaces as a normal tool error (the server turns `__blocked` into an error result). If a plugin blocks a tool you expected to succeed, do not retry it blindly; report the block reason and pick another approach.
- An `onAfterAction` handler can rewrite a tool's result. Treat the returned payload as authoritative, not a fixed schema.
- Plugins never inject plaintext secrets into error output; the redaction layer (`SessionSecretTracker.redactAll`) runs on all error strings.
