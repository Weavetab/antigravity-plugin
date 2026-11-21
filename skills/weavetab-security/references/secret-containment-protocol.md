---
id: secret-containment-protocol
domain: security
triggers:
  - "enter password"
  - "input credentials"
  - "api key entry"
  - "sensitive token"
  - "secret containment"
tools:
  - "browser_type_secret"
weavetab: ">=2.6.4"
---

# Secret Containment Protocol & Zero-Bleed Execution

AI agent transcripts, LLM providers, and tool logs are high-risk attack surfaces for credential leaks. Weavetab enforces **Zero-Bleed Secret Containment** for sensitive inputs.

---

## 1. Entering Credentials with `browser_type_secret`

Whenever an agent inputs passwords, two-factor backup codes, credit card numbers, or API keys, you MUST use `browser_type_secret`.

`browser_type_secret` never accepts a plaintext secret and never reads raw `process.env`. Secrets are stored in Weavetab's domain-scoped secret store (`JsonSecretStore`) via the CLI so the value stays in-process and never enters the agent transcript. Store the credential first:

```bash
weavetab secrets set ACCOUNT_PASSWORD
```

Then reference it by `envKey`:

```json
{
  "ref": "w:12",
  "envKey": "ACCOUNT_PASSWORD"
}
```

Accepted targeting parameters: `id` (alias `ref`) or `target` (CSS selector). `envKey` is **required**.

---

## 2. Domain & URL Origin Enforcement

Before loading the secret value from the store, `browser_type_secret` inspects the active tab's URL:

1. **No-Host URLs Blocked (`INVALID_URL`)**: Local and synthetic origins without a hostname (such as `file://` and `about:blank`) are rejected immediately.
2. **Allowed-Domain Check (`DOMAIN_NOT_ALLOWED`)**: If the current tab's hostname is not in the secret key's allowed domain list, the call hard-fails before the secret is resolved. This prevents prompt-injected navigations from exfiltrating credentials to attacker origins.
3. **Missing Key (`SECRET_NOT_FOUND`)**: If the key has not been configured, instruct the user to run `weavetab secrets set <KEY>`.

---

## 3. The Zero-Bleed Containment Architecture

When `browser_type_secret` executes:
1. **In-Process CDP Injection**: The secret flows directly from `secretStore` to CDP `Input.insertText` without crossing tool arguments or results.
2. **DOM & HUD Hardening**: The target input is marked `data-wt-secret="true"` (redacted by subsequent `browser_map`, `browser_scrape`, `browser_snapshot`, and delta reads) and the HUD displays `injecting [REDACTED]...`.
3. **Log & Result Sanitization**: Trail entries, server logs, error messages (`SessionSecretTracker.redactAll`), and tool outputs strip the secret string and return only metadata:
   ```json
   {
     "success": true,
     "typed_into": "Password",
     "ref": "w:12",
     "key": "ACCOUNT_PASSWORD",
     "domain": "portal.example.com"
   }
   ```
4. **No Re-Emission**: Never print or echo raw credentials into subsequent model thoughts or conversation messages.

---

## 4. Strict Operating Prohibition

> [!CAUTION]
> Under NO circumstances should an agent pass passwords or credentials to:
> - `browser_type`
> - `browser_fill`
> - `browser_eval`
>
> Doing so causes plaintext secrets to enter unmasked transcripts and tool logs.
