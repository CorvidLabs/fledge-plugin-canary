---
id: CHG-0001-adopt-specsync-5-0-1-and-trust-1-0-0-governance-for-the-native-canary-fledge-plu
state: accepted
type: migration
base_commit: d16a8e6fd794e38c295db2541ec5b6ab54c5906b
---

# Adopt SpecSync 5.0.1 and Trust 1.0.0 governance for the native Canary Fledge plugin

## Intent

Adopt SpecSync 5.0.1 and Trust 1.0.0 governance for the native Canary Fledge plugin

## Affected Canonical Specs

- None

## Acceptance Criteria

- SpecSync strict check passes at explicit advisory threshold 0; all four integrations report installed; Trust doctor and verification pass; ShellCheck validates native and legacy canaries without weakening security semantics

## No-spec Rationale

This governance adoption documents existing Canary outcomes and verification policy without changing security or runtime semantics.
