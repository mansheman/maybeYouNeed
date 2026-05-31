2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[JWT]]
###### Prasyarat: [[JSON Web Tokens (JWT)]]
# JWT Structure

## Header

Metadata tentang algoritma dan tipe token:

```json
{"alg":"HS256","typ":"JWT"}
```

## Payload

Claims tentang subject/session:

```json
{"sub":"123","name":"John Doe","admin":true,"exp":1716454560}
```

## Signature

Mencegah token diubah (dibuat dengan menandatangani header+payload menggunakan key).

Poin penting:

- kalau header/payload berubah, verifikasi signature harus gagal

---

## Diagram

![[Pics/auth/jwt-structure.svg]]
