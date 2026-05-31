2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[2FA]]
###### Prasyarat: [[Two-Factor Authentication (2FA)]]
# 2FA Testing Methodology

## Pengumpulan informasi

- identifikasi mekanisme 2FA (SMS/email/TOTP/app/hardware)
- pahami proses enrollment dan recovery
- review konfigurasi: panjang token, masa berlaku, rate limiting

---

## Uji flow autentikasi

- kekuatan OTP (panjang/entropy)
- expiry OTP (time-based + invalid setelah dipakai)
- replay attempt (OTP yang sama dipakai dua kali)
- pastikan transport aman (HTTPS)

---

## Rate limiting dan lockout

- ketahanan brute force (rate limit / temporary lock)
- perbedaan response yang membocorkan validitas (status/error message)

---

## Area lanjutan

- bypass di integrasi OAuth/OpenID
- lokasi validasi token (harus server-side)
