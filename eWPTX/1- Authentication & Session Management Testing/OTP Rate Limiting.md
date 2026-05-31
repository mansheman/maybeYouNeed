2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[2FA]]
###### Prasyarat: [[OTP Security]]
# OTP Rate Limiting

Rate limiting OTP mencegah brute force dan penyalahgunaan dengan membatasi jumlah percobaan verifikasi dalam jendela waktu tertentu.

Yang dicek:

- per-user and per-IP throttling
- progressive delays
- temporary lockout after N failures
- monitoring/alerting
