2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[JWT]]
###### Prasyarat: [[JSON Web Tokens (JWT)]]
# JWT Claims

## Apa itu claims?

Claims adalah pasangan key-value di payload JWT yang memberi konteks untuk autentikasi/otorisasi (mis. siapa user-nya, role apa, kapan token kadaluarsa).

Claims **tidak terenkripsi** secara default.

---

## Jenis claims

### Registered claims (standar)

- `iss` issuer
- `sub` subject
- `aud` audience
- `exp` expiration
- `iat` issued at
- `nbf` not before

### Public claims (ditentukan aplikasi)

Examples:

- `role`
- `email`
- `permissions`

### Private claims (disepakati antar pihak)

Examples:

- `department`
- `cartId`
