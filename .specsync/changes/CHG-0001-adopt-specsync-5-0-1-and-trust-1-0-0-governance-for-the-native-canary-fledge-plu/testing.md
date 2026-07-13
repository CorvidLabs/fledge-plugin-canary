---
change: CHG-0001-adopt-specsync-5-0-1-and-trust-1-0-0-governance-for-the-native-canary-fledge-plu
artifact: testing
---

# Testing

- `shellcheck bin/canary bin/canary-legacy`
- `specsync check --strict --force` at advisory threshold 0
- `specsync agents status`
- `fledge trust doctor`
- `fledge trust verify`
- Independent native/WASM boundary runs remain specialized security checks
