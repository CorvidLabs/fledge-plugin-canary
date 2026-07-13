---
change: CHG-0005-avoid-embedding-raw-gemini-command-arguments-in-literal-shell-syntax-when-creati
artifact: testing
---

# Testing

- Inspect the generated prompt with an intent containing embedded quotes.
- Confirm the guidance does not place raw `{{args}}` inside literal shell syntax.
- Run strict SpecSync validation and the native Canary verification lane.
