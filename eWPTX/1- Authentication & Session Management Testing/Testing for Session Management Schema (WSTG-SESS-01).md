2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[Session Management]]
###### Prasyarat: [[Session Management Testing]]
# Testing for Session Management Schema (WSTG-SESS-01)

## Gambaran Singkat

Tes OWASP WSTG **WSTG-SESS-01** fokus untuk menilai keamanan **manajemen sesi** pada aplikasi web.

Tujuan praktisnya: memastikan skema session (termasuk **session ID** dan **cookie**) didesain dengan kuat dan tidak mudah dipalsukan/diambil alih.

---

## Apa yang divalidasi

- bagaimana token sesi **dibuat** (entropy/acaknya cukup atau tidak)
- bagaimana sesi **dipelihara** (rotasi/regenerasi di momen penting)
- bagaimana sesi **diakhiri** (logout benar-benar invalid di server)
- apakah aplikasi mempercayai **data token di sisi client** untuk keputusan otorisasi

---

## Pola serangan yang sering

1) kumpulkan cookie (ambil sample yang cukup)
2) reverse engineering cookie (pahami struktur/encoding)
3) manipulasi cookie (forge/tamper)

Tergantung cara cookie dibuat, proses ini bisa butuh banyak percobaan (mis. brute force) sebelum terlihat pola/kelemahannya.

---

## Objektif pengujian (ringkas)

- kumpulkan token sesi dari user yang sama dan user berbeda
- analisis randomness (seberapa tahan terhadap forgery)
- uji modifikasi cookie yang tidak ditandatangani/masih bisa diubah, apalagi yang memuat data keamanan

---

## Tes/teknik kunci

|Tes/Teknik|Yang dilakukan|Risiko jika lemah|
|---|---|---|
|Session ID dapat diprediksi|analisis randomness token|menebak/memalsukan sesi|
|Session fixation|paksa/pertahankan session ID yang sudah diketahui sebelum login|takeover setelah login|
|Expiry/termination|verifikasi timeout + invalidasi saat logout|replay/reuse sesi|
|Session hijacking|replay token yang dicuri|pengambilalihan akun|
|Flag keamanan cookie|cek `HttpOnly`, `Secure`, `SameSite`|risiko XSS/CSRF/MitM|
|Token bocor di URL|pastikan tidak ada session ID di URL|bocor via log/referrer/history|
