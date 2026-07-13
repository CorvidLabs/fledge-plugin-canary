---
spec: canary.spec.md
---

## User Stories

- As a security maintainer, I want capability bypasses to fail loudly.
- As a plugin author, I want an honest report of ambient native-process exposure compared with WASM.

## Acceptance Criteria

### REQ-canary-001

The canary SHALL verify metadata, exec, and store RPC behavior against the capabilities actually granted.

### REQ-canary-002

The canary SHALL separately probe and report ambient native environment, filesystem, network, persistence, and process access.

### REQ-canary-003

The canary SHALL mask sensitive values and distinguish PASS, FAIL, WARN, BLOCKED, and LEAKED outcomes.

### REQ-canary-004

The canary SHALL fail when a protected capability boundary is bypassed while retaining expected warnings about unsandboxed native access.

### REQ-canary-005

Legacy mode SHALL report inherited environment and filesystem visibility without requesting protocol capabilities.

## Constraints

- The native plugin is intentionally unsandboxed and must not be interpreted as providing WASM isolation.

## Out of Scope

- Exploitation, secret exfiltration, or modification of discovered sensitive data.
