2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]]
###### Prerequisites: [[Token-Based Authentication]]
# Token Placement

## Authorization header (recommended)

```http
GET /api/v1/resource HTTP/1.1
Host: example.com
Authorization: Bearer <token>
```

Benefits:

- standard for APIs
- avoids URL leakage

---

## Query parameters (avoid)

```http
GET /api/v1/resource?access_token=<token> HTTP/1.1
```

Risks:

- leaks via history, logs, referrers
- may be cached

---

## Request body

Used for POST flows, but avoid placing tokens in GET bodies.

---

## Cookies

```http
Set-Cookie: auth_token=<token>; Secure; HttpOnly; SameSite=Lax
```

Pros:

- automatic inclusion for same-site requests

Risks:

- CSRF if not protected

---

## Custom headers

```http
X-Auth-Token: <token>
```

Less standard than `Authorization`.
