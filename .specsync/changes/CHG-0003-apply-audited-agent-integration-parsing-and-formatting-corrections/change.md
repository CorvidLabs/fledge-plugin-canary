---
id: CHG-0003-apply-audited-agent-integration-parsing-and-formatting-corrections
state: implementing
type: documentation
base_commit: f597526a8e16108e34c77776a0ecc39890c30c80
---

# Apply audited agent-integration parsing and formatting corrections

## Intent

Apply audited agent-integration parsing and formatting corrections

## Affected Canonical Specs

- None

## Acceptance Criteria

- All three create-spec command integrations explicitly trim surrounding input whitespace and all four skills format the requirements.md companion name consistently.

## No-spec Rationale

The corrections only clarify generated agent command parsing and Markdown file-name formatting; canonical Canary behavior and requirements do not change.
