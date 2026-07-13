---
change: CHG-0003-apply-audited-agent-integration-parsing-and-formatting-corrections
artifact: context
---

# Context

Hosted review of the accepted reverification found two consistency gaps in generated agent guidance: removing `--minimal` did not explicitly trim surrounding whitespace, and skill prose named the requirements companion without Markdown code formatting. These files guide agents only; native Canary executables and canonical Canary requirements are unchanged.
