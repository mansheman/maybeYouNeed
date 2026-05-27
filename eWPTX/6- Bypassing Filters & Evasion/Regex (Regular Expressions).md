2026-02-05 06:17

Status:

Tags: [[eWPTX]]
###### Prerequisites: [[Bypassing Filters & Evasion]]
# Regex (Regular Expressions)

## Overview

**Regex** is a pattern language used to match (and sometimes replace) text.

In security, regex shows up in:

- WAF rules
- input validation filters
- log parsing/detection
- allow-lists / deny-lists

---

## Why it matters for bypassing filters

A lot of “filters” are just regex patterns.

If you understand the regex, you can predict:

- what it blocks
- what it allows
- where the gaps are (anchors, greediness, case sensitivity, Unicode, etc.)

---

## Quick basics

### Common tokens

- `.` any character (except newline in many engines)
- `*` zero or more
- `+` one or more
- `?` optional (zero or one)
- `{m,n}` repetition range
- `^` start of string/line
- `$` end of string/line
- `|` OR
- `()` group / capture
- `(?:...)` non-capturing group
- `[]` character class
  - `[abc]` one of a/b/c
  - `[a-z]` range
  - `[^a-z]` NOT range
- `\d` digit, `\w` word char, `\s` whitespace (engine-dependent details)

### Examples

Match an email-ish pattern (simplified):

```regex
^[^@\s]+@[^@\s]+\.[^@\s]+$
```

Match `admin` or `root` (case-insensitive flag is usually external):

```regex
^(admin|root)$
```

Match anything that contains `select` (naive SQLi keyword filter):

```regex
select
```

---

## Testing a filter quickly (recommended)

Copy the regex and paste it into regex101, then it will explain the pattern and show matches.

- Site: https://regex101.com

Tips when using it:

- choose the correct regex “flavor” (PCRE / Python / JavaScript)
- test both allowed and blocked inputs
- look for missing anchors (`^`/`$`) and overly broad patterns

---

## Common regex mistakes that create bypasses

- no anchors: pattern matches a substring, not the whole input
- catastrophic backtracking: regex DoS risk
- using deny-lists instead of allow-lists
- inconsistent normalization (URL decode, Unicode, case folding) between filter and backend
