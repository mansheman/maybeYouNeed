2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[JWT]]
###### Prasyarat: [[Token-Based Authentication]]
# JSON Web Token (JWT)

## Gambaran Singkat

JWT adalah token yang ringkas dan URL-safe untuk kebutuhan **autentikasi/otorisasi** dan pertukaran informasi. JWT sering dipakai di API karena mudah dikirim lewat header.

Struktur umum:

```text
HEADER.PAYLOAD.SIGNATURE
```

- **header**: tipe token + algoritma yang dipakai
- **payload**: kumpulan *claims* (data user/session)
- **signature**: memastikan token tidak diubah (integrity/authenticity)

Hal penting:

- payload itu **di-Base64-encode**, bukan dienkripsi → orang bisa membaca isinya.
- jangan taruh rahasia (mis. password/API key) di claims JWT.
