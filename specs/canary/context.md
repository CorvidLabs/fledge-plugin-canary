---
spec: canary.spec.md
---

## Context

This first-party audit tool verifies documented differences between capability-gated RPC access and ambient access available to a native subprocess.

## Related Modules

- fledge plugin protocol and capability runtime
- fledge-plugin-canary-wasm

## Design Decisions

- Report expected native exposure honestly instead of treating all readable resources as a test failure.
- Never print full secret values.
