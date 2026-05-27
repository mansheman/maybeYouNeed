2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]]
###### Prerequisites: [[Token Placement]]
# Token Placement - Best Practices

- Prefer `Authorization: Bearer` for APIs.
- Avoid query parameter tokens unless unavoidable.
- If using cookies: set `HttpOnly`, `Secure`, and appropriate `SameSite`.
- Always use HTTPS.
- Minimize token exposure (avoid places that get logged/cached).
