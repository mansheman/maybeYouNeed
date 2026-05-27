2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[Session Management]]
###### Prerequisites: [[Session Management Testing]]
# Testing for Session Management Schema (WSTG-SESS-01)

## Overview

The OWASP WSTG test **WSTG-SESS-01** focuses on assessing the security of session management in web applications.

Goal: validate the design and robustness of the session management schema, including **session IDs** and **cookies**.

---

## What you’re validating

- how session tokens are **created** (entropy/randomness)
- how sessions are **maintained** (rotation/regeneration)
- how sessions are **terminated** (logout/invalidation)
- whether the app relies on **client-side token data** for authorization

---

## Common attack pattern

1) cookie collection (capture enough samples)
2) cookie reverse engineering (understand structure/encoding)
3) cookie manipulation (forge/tamper)

Depending on how the cookie is created, this may require many attempts (brute force).

---

## Test objectives (abridged)

- gather session tokens for the same user and different users
- analyze randomness (resistance to forging)
- modify unsigned/modifiable cookies that contain security-relevant data

---

## Key tests / techniques

|Test/Technique|What you do|Risk if weak|
|---|---|---|
|Predictable session IDs|analyze token randomness|session guessing/forging|
|Session fixation|force/keep a known session ID before login|takeover after login|
|Expiration/termination|verify timeout + logout invalidation|replay/reuse|
|Session hijacking|replay a stolen token|account takeover|
|Cookie security flags|check `HttpOnly`, `Secure`, `SameSite`|XSS/CSRF/MitM risk|
|Token exposure in URLs|ensure no session IDs in URLs|leak via logs/referrers/history|
