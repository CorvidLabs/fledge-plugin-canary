---
id: CHG-0005-avoid-embedding-raw-gemini-command-arguments-in-literal-shell-syntax-when-creati
state: implementing
type: bug_fix
base_commit: 41021dd288507888a55513bca7e2a42f6eb6af77
---

# Avoid embedding raw Gemini command arguments in literal shell syntax when creating an SDD change

## Intent

Avoid embedding raw Gemini command arguments in literal shell syntax when creating an SDD change

## Affected Canonical Specs

- None

## Acceptance Criteria

- Gemini guidance treats raw arguments as intent data rather than literal shell syntax; quoted intent cannot produce a malformed suggested command; strict SpecSync and native verification pass

## No-spec Rationale

This corrects generated agent command safety without changing Canary runtime behavior or its canonical native plugin contract.
