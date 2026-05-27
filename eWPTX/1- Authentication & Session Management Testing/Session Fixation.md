2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[Session Management]]
###### Prerequisites: [[Session Management Testing]]
# Session Fixation

## Definition

Session fixation happens when an attacker forces a victim to use a session ID the attacker already knows, then reuses it after the victim authenticates.

---

## How it works (high level)

1) attacker sets/acquires a session ID
2) victim logs in while using that same session ID
3) app associates that session ID with the authenticated session
4) attacker reuses the known session ID to access the victim session

---

## Common causes

- not regenerating session ID on login
- allowing session IDs in URLs or client-controlled locations
- weak/predictable session ID generation
- missing cookie flags (`Secure`, `HttpOnly`, `SameSite`)
- long-lived sessions without expiry/timeout

---

## Testing ideas

- log in and check whether session ID changes after authentication
- test whether the app accepts session IDs in URL/query
- verify logout invalidates the session server-side
