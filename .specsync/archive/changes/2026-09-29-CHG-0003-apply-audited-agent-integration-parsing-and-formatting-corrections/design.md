---
change: CHG-0003-apply-audited-agent-integration-parsing-and-formatting-corrections
artifact: design
---

# Design

Keep the Claude, Cursor, and Gemini create-spec instructions behaviorally equivalent: remove `--minimal`, trim only leading and trailing whitespace, then classify the complete remaining input. Keep all four skills textually equivalent and render `requirements.md` as a file name.
