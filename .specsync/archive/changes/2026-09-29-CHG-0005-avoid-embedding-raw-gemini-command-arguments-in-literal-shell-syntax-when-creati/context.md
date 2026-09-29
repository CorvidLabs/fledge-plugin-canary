---
change: CHG-0005-avoid-embedding-raw-gemini-command-arguments-in-literal-shell-syntax-when-creati
artifact: context
---

# Context

Gemini substitutes `{{args}}` as raw prompt text. Embedding that value inside a
literal quoted command can produce malformed shell syntax when an intent itself
contains quotes. The generated guidance must treat the value as data and require
safe single-argument handling.
