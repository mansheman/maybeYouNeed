2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[JWT]]
###### Prerequisites: [[JSON Web Tokens (JWT)]]
# JWT Claims

## What are claims?

Claims are key-value pairs in the JWT payload used to provide context for authentication/authorization.

Claims are not encrypted by default.

---

## Types of claims

### Registered claims (standard)

- `iss` issuer
- `sub` subject
- `aud` audience
- `exp` expiration
- `iat` issued at
- `nbf` not before

### Public claims (app-defined)

Examples:

- `role`
- `email`
- `permissions`

### Private claims (agreed between parties)

Examples:

- `department`
- `cartId`
