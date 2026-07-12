---
module: canary
version: 1
status: active
files:
  - bin/canary
  - bin/canary-legacy

db_tables: []
depends_on: []
---

# Canary

## Purpose

Audit fledge plugin security boundaries by proving both the fledge-v1 capability denials and the ambient access retained by unsandboxed native subprocesses, with a legacy environment-exposure mode for comparison.

## Public API

| Section | Behavior |
|---------|----------|
| all | Run capability-denial checks plus native baseline exposure probes. |
| metadata | Verify metadata access follows the granted capability and filters sensitive variables. |
| exec | Verify command execution denial or document granted shell reach. |
| store | Verify storage denial and key/value size boundaries. |
| baseline | Probe direct environment, filesystem, network, persistence, clipboard, and process access. |
| expose | Produce the detailed masked exposure report. |
| legacy | Report inherited environment and filesystem visibility without relying on capability RPCs. |

## Invariants

1. The plugin declares zero capabilities; capability RPCs must be denied unless explicitly granted by a test harness.
2. Direct native-process exposure is reported as WARN or LEAKED evidence, not misclassified as an RPC capability failure.
3. Secret values are masked in reports; only names and limited prefixes may be shown.
4. Boundary violations that terminate the process count as enforced boundaries.
5. The report distinguishes native subprocess access from the companion WASM sandbox guarantees.
6. Any genuine capability bypass increments FAIL and causes a non-zero result.

## Behavioral Examples

```
Given the native canary is granted no exec capability
When it requests an exec RPC
Then code 126 is reported as PASS while separately observable ambient shell access is reported as WARN
```

## Error Cases

| Error | When | Behavior |
|-------|------|----------|
| Capability bypass | A denied RPC returns protected data or executes work | Record FAIL and surface the regression. |
| Missing optional tool | A network, clipboard, or persistence probe is unavailable | Report the boundary as unavailable without inventing leakage. |
| Boundary termination | Fledge closes the plugin during an oversized storage probe | Treat termination as successful enforcement. |
| Malformed or missing init | Required capability/project data cannot be decoded | Exit rather than claim a passing boundary. |

## Dependencies

- Bash and jq
- fledge-v1 plugin protocol
- optional host tools used only to detect ambient native access
- fledge-plugin-canary-wasm as the sandboxed comparison

## Change Log

| Version | Date | Changes |
|---------|------|---------|
| 1 | 2026-07-12 | Document existing native canary security semantics for SpecSync 5 adoption. |
