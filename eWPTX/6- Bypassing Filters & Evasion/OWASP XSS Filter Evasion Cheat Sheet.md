2026-02-05 06:25

Status:

Tags: [[eWPTX]] [[XSS Attacks]]
###### Prerequisites: [[Bypassing Filters & Evasion]]
# OWASP XSS Filter Evasion Cheat Sheet

## Reference

- OWASP XSS Filter Evasion Cheat Sheet: https://owasp.org/www-community/xss-filter-evasion-cheatsheet

> This OWASP page contains many payload variations. Use only in **authorized labs/assessments**.

---

## Why this exists

Many applications try to stop XSS using **blacklists** (e.g., blocking `<script>`), weak regex filters, or partial encoding.

Attackers then look for:

- a different **context** (HTML, attribute, JS string, URL, CSS)
- a different **parser behavior** (browser parsing quirks, normalization/decoding order)
- a different **injection primitive** (event handlers, SVG/XML, URL schemes, etc.)

---

## Defensive takeaways (what to do instead of filter games)

- Use **context-aware output encoding** (HTML, attribute, JS, URL, CSS contexts).
- Prefer **allow-lists** over deny-lists for inputs where possible.
- Apply **normalization once** (decode/canonicalize), then validate, then encode on output.
- Use frameworks/templates that auto-escape and don’t disable it.
- Add a strong **CSP** (Content Security Policy) as a mitigation layer.
- Avoid dangerous sinks (`innerHTML`, `document.write`, unsafe DOM APIs) unless strictly necessary and safe.

---

## When analyzing a regex “filter”

If a filter is implemented as a regex, copy it and test it in regex101 (pick the right flavor) to understand what it matches:

- https://regex101.com

Look for common mistakes:

- missing anchors (`^` / `$`) → substring matches
- case-sensitivity assumptions
- inconsistent decoding (URL decode / HTML entities / Unicode normalization)

