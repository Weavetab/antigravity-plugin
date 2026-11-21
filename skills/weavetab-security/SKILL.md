---
name: weavetab-security
description: Enforce secret-safe credential injection (browser_type_secret), domain allowlists/denylists, and RBAC permission boundaries in Weavetab MCP. Use whenever handling passwords, API keys, tokens, logins, or restricted environments.
---

# Weavetab Security & Secret Containment

Read the matching security playbook before entering sensitive data or configuring policy boundaries:

- **Blind credential & secret injection**: Read [references/secret-containment-protocol.md](references/secret-containment-protocol.md) (`browser_type_secret`).
- **RBAC roles and domain policies (`~/.weavetab/config.json`)**: Read [references/rbac-and-policies.md](references/rbac-and-policies.md).

## Hard Rule

Always pass passwords, API keys, and sensitive tokens through `browser_type_secret` so inputs are sanitized to `[REDACTED]`. Never pass secrets to `browser_type` or `browser_fill`.
