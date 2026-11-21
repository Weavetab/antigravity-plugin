---
name: weavetab-resilience
description: Recover cleanly from detached React/Next/Vue DOM refs, save and restore browser checkpoints, break infinite interaction loops, and pause on CAPTCHAs in Weavetab MCP workflows. Use when an automation hits stale refs, hydration races, loops, or risky multi-step state changes.
---

# Weavetab Resilience & Recovery

Read the reference guide matching the failure mode:

- **Detached elements & SPA re-render recovery**: Read [references/stale-ref-recovery.md](references/stale-ref-recovery.md).
- **Save/restore checkpoints & rollback failed flows**: Read [references/session-time-travel.md](references/session-time-travel.md) (`browser_checkpoint`).
- **Break interaction loops and handle CAPTCHA gates safely**: Read [references/loop-breaker.md](references/loop-breaker.md) (`browser_map`, `browser_captcha`).

Never attempt to bypass or solve CAPTCHAs automatically; detect the gate and ask the user to complete it in their live window.
