2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]]
###### Prasyarat: [[Token-Based Authentication]]
# Penempatan Token

## Header Authorization (disarankan)

```http
GET /api/v1/resource HTTP/1.1
Host: example.com
Authorization: Bearer <token>
```

Kelebihan:

- standar de-facto untuk API
- menghindari token bocor lewat URL

---

## Query parameter (sebaiknya dihindari)

```http
GET /api/v1/resource?access_token=<token> HTTP/1.1
```

Risiko:

- mudah bocor via browser history, access log, dan `Referer`
- bisa ikut ke-cache

---

## Request body

Bisa dipakai pada flow POST tertentu, tapi hindari “mengakali” dengan menaruh token di body untuk request GET.

---

## Cookie

```http
Set-Cookie: auth_token=<token>; Secure; HttpOnly; SameSite=Lax
```

Kelebihan:

- otomatis ikut terkirim pada request same-site

Risiko:

- rentan CSRF jika tidak ada mitigasi yang benar

---

## Custom header

```http
X-Auth-Token: <token>
```

Lebih tidak standar dibanding `Authorization`.
