---
id: CHG-0001-adopt-specsync-5-0-1-and-trust-1-0-0-governance-for-the-native-canary-fledge-plu
state: archived
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

## Migration Note

Migrated by hand to SpecSync 6 per Leif's decision (2026-09-28); the 6.0.0 tool refused to archive this legacy record (`` legacy accepted change `CHG-0001-adopt-specsync-5-0-1-and-trust-1-0-0-governance-for-the-native-canary-fledge-plu` requires exactly one distinct valid historical reconstruction, found 0; first reconstruction failure: legacy accepted change `CHG-0001-adopt-specsync-5-0-1-and-trust-1-0-0-governance-for-the-native-canary-fledge-plu` cannot reproduce its signed raw-content aggregate ``).

- Workflow v1 (SpecSync 5) record, accepted on 2026-07-13 by the closing approval already stored in `approvals.json`. SpecSync 6.0.0 reports its accepted evidence as stale, for the reason quoted above.
- Moved by hand from `.specsync/changes/CHG-0001-adopt-specsync-5-0-1-and-trust-1-0-0-governance-for-the-native-canary-fledge-plu/` into the layout `specsync change archive` writes: `accepted-state.json` is the unchanged accepted `state.json`, `state.json` is marked `archived`, and this file's front matter says `archived`.
- `approvals.json`, `verification.json` and every other artifact are the original SpecSync 5 evidence, unchanged. `verification.json` verifies commit `fe2b56c29934d2f7d1831c4ad0c29b864d6e3d84`, not the tree this record was archived from.
- There is no `verification-attempts.json`: SpecSync 5 did not write one for this record, and this migration does not invent attempt history.
- This migration added no verification evidence, test result, attempt history, or approval. It is a manual migration, not a fresh re-verification.
- Closing it through the tool takes `specsync change reopen`, `specsync change verify`, then `specsync change accept`, which writes a new closing approval. Per Leif's decision it was archived by hand instead, so no reopen or new approval is recorded.
