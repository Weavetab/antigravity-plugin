---
id: rbac-and-policies
domain: security
triggers:
  - "rbac"
  - "security policy"
  - "role boundaries"
  - "viewer agent"
  - "permission denied"
tools:
  - "browser_inspect"
  - "browser_scrape"
  - "browser_map"
weavetab: ">=2.6.4"
---

# Role-Based Access Control (RBAC) & Agent Guardrails

Weavetab allows developers to run untrusted or lower-privilege subagents within strict security boundaries to prevent accidental destructive operations or data leaks.

Roles are configured per agent (in `config.json`). The server checks the role **before** launching a browser or resolving the target.

---

## 1. The 5 Security Roles

| Role | Tool Access | Intended Use |
| :--- | :--- | :--- |
| `admin` | Full access (`*`). | Primary operator / lead agent. |
| `operator` | Full access (`*`). | Standard automation agent. |
| `automation` | Every tool except `browser_thoughts` (automation agents must be non-interactive). | Background cron / worker. |
| `viewer` | Read-only set only (13 tools): `browser_map`, `browser_find`, `browser_scrape`, `browser_console`, `browser_inspect`, `browser_snapshot`, `browser_screenshot`, `browser_pdf`, `browser_cookies`, `browser_tabs`, `browser_wait`, `browser_pattern_learn`, `browser_detect`. `browser_eval` is excluded. | Research subagents / scrapers. |
| `system` | `browser_thoughts`, `browser_detect`, `browser_performance`, `browser_checkpoint`, `browser_pattern_learn`. | Internal/system operations. |

Any unknown or unrecognized role value is rejected (`Unknown or unauthorized role`).

---

## 2. Action-Level Read-Only Enforcement

Both the `viewer` role and domains configured with `mode: "read-only"` enforce action-level restrictions on dual-purpose tools:

| Tool | Permitted Actions / Constraints in `viewer` and `read-only` Mode |
| :--- | :--- |
| `browser_cookies` | Only `action: "get"` or `"list"`. If `url` is supplied, it must be **same-origin** with the active tab and pass domain policy. |
| `browser_tabs` | Only `action: "list"` or `"list_windows"` (opening, closing, switching, or focusing windows is blocked). |
| `browser_snapshot` | Only `action: "capture"` (`"restore"` is blocked). `filePath`/`filename` rejects `..` path traversal, and absolute paths must stay inside the configured `snapshotDir`. |
| `browser_console` | Only `action: "read"` (`"clear"` is blocked). |
| `browser_pattern_learn` | Only `action: "get"` (`"learn"`, `"record_execution"`, and `"record_quirk"` are blocked). |

---

## 3. Handling Permission Denials

There is no JSON `error_code` for RBAC. A denied call fails with `isError: true` and a plain-text message prefixed with `[!]`:

```
[!] Role 'viewer' does not have permission to use 'browser_click'. Viewer role only permits read-only tools (map/find/scrape/console/inspect/snapshot/screenshot/pdf/cookies/tabs/wait/pattern_learn/detect).
```

### Protocol:
1. Do not retry the denied tool; the role will not change mid-session.
2. Delegate the interactive action to the parent agent or a higher-privilege operator.
3. Limit your output to findings gathered via read-only tools (`browser_scrape`, `browser_map`, `browser_inspect`).
