# Weavetab Plugin for Antigravity

Local-first browser and desktop automation for AI agents, bundling **Weavetab MCP** (`@weavetab/mcp`), the full **Weavetab Skills** (`@weavetab/skills` v2.6.4) playbook suite, and operational guardrails into one Antigravity plugin.

Versioned in lockstep with `@weavetab/mcp`, `@weavetab/sdk`, and `@weavetab/skills` (`v2.6.4`).

## What's Included

- **MCP Server (`weavetab`)**: Launches `@weavetab/mcp@latest` over local stdio (`npx -y @weavetab/mcp@latest`), exposing 40 browser and Electron desktop automation tools over direct Chrome DevTools Protocol (CDP).
- **Full `@weavetab/skills` Suite (9 Domains)**: Progressive-disclosure `SKILL.md` entrypoints backed by all 22 operational reference guides:
  - `weavetab-browser`: 4-tier token-efficient DOM reading (`browser_find`, `browser_map` with `lite`/`delta`), interaction pipelines, multi-field forms, and tab management.
  - `weavetab-desktop`: Electron desktop app discovery and targeting (`Discord`, `Slack`, `VS Code`, `Figma`, `Obsidian`, `Notion`, `Postman`, `Spotify`, and local apps).
  - `weavetab-memory`: Selector cache, site workflow learning (`browser_pattern_learn`), and compiled `browser_burst` macros.
  - `weavetab-resilience`: Detached SPA ref recovery, `browser_checkpoint` rollback, loop breaking, and safe CAPTCHA pausing.
  - `weavetab-security`: Blind credential injection (`browser_type_secret`), RBAC roles, and domain policies.
  - `weavetab-plugins`: Discovering, running, and authoring `@weavetab/sdk` plugins, hooks, and bounded `AGENTS.md` playbooks.
  - `weavetab-network-perf`: Network request interception/mocking, Core Web Vitals, wait conditions, and device emulation.
  - `weavetab-documents`: `.docx` / `.xlsx` / `.pdf` / `.csv` ingestion, PDF export, canvas pixel capture, and visual screenshots.
  - `weavetab-developer`: Repository forensics and issue/PR workflows with `@weavetab/plugin-github`.
- **Rules (`rules/AGENTS.md`)**: Session handshake, token conservation, `w:NN` ref discipline, credential safety, desktop app targeting, and CAPTCHA boundaries.

## Prerequisites

- **Node.js** `20.19+` (or `22.12+` / `23+`) with `npx` available on your `PATH`.
- A local Chromium-based browser (Chrome, Edge, Brave, Arc) or Electron desktop app with CDP enabled.

## Installation

Clone and install with the Antigravity CLI:

```bash
git clone https://github.com/Weavetab/antigravity-plugin.git
agy plugin install ./antigravity-plugin
```

Verify installation:

```bash
agy plugin list
```

Toggle the plugin or inspect its components anytime under **Customizations -> Installed** in the Antigravity app or IDE.

### Latest Package Tags

All runtime & CLI commands track `@latest` automatically:

- MCP Server (bundled in `mcp_config.json`): `npx -y @weavetab/mcp@latest`
- Agent Skills CLI: `npx -y @weavetab/skills@latest`
- MCP Plugin SDK: `npm install -D @weavetab/sdk@latest`

## Example Prompts

- `"Detect my active browsers and Electron desktop apps with Weavetab"`
- `"Map the active tab in lite mode and fill out the form safely without leaking credentials"`
- `"List my open Discord windows with Weavetab and inspect the active channel"`
- `"Capture a full-page screenshot and export the current page to PDF"`

## Links

- Website: https://weavetab.dev
- Weavetab MCP: https://github.com/Weavetab/mcp
- Weavetab Skills: https://github.com/Weavetab/skills
- MCP Plugin SDK (`@weavetab/sdk`): https://weavetab.dev/mcp/sdk
- Plugin Registry: https://weavetab.dev/discover
- Support: https://weavetab.dev/contact

## License

MIT. See [LICENSE](LICENSE). The `@weavetab/mcp` server launched via `mcp_config.json` is licensed separately under `AGPL-3.0-only`.
