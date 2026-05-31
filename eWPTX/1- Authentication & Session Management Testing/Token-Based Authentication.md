2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]]
###### Prasyarat: [[Authentication]]
# Autentikasi Berbasis Token

## Gambaran Singkat

Autentikasi berbasis token adalah metode untuk memvalidasi identitas dan/atau otorisasi user/aplikasi menggunakan **token** yang selalu dikirim client pada setiap request.

Dibanding session klasik yang disimpan server-side, pendekatan token sering lebih **stateless** (terutama untuk API), sehingga lebih mudah diskalakan—dengan trade-off: token harus dijaga ketat karena “siapa pun yang memegang token” bisa menyamar sebagai user.

---

## Jenis Token (umum)

### Bearer tokens

- siapa pun yang memegang token bisa menggunakannya
- sangat umum di API
- wajib dilindungi dan idealnya berumur pendek

### JSON Web Tokens (JWT)

- token “self-contained”: header + payload (claims) + signature
- banyak dipakai untuk autentikasi stateless

### OAuth tokens

- access token / refresh token
- dipakai di flow OAuth2 untuk otorisasi
