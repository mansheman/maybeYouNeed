2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[2FA]]
###### Prasyarat: [[Two-Factor Authentication (2FA)]]
# Keamanan OTP

## Gambaran Singkat

OTP adalah kode sementara yang umumnya sekali pakai untuk memverifikasi identitas saat login/aksi sensitif.

Properti keamanan yang ideal:

- masa berlaku singkat
- sekali pakai
- ada pembatasan percobaan (rate limit)

---

## Metode OTP

- TOTP (berbasis waktu)
- SMS OTP
- Email OTP

---

## Kontrol yang bagus

- window expiry yang ketat
- invalid setelah dipakai
- rate limiting dan lockout
- pengiriman aman (HTTPS, hindari downgrade)
