2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[2FA]]
###### Prasyarat: [[Two-Factor Authentication (2FA)]]
# Bypass 2FA

## Kategori bypass yang umum (untuk pengujian)

### Social engineering

- phishing untuk password + OTP
- vishing/smishing

### Implementation flaws

- recovery flow lemah (security question, backup code yang buruk)
- session fixation di boundary sebelum/sesudah 2FA
- validasi token buruk (replay, tidak ada expiry)

### Token interception

- MitM / downgrade kalau tidak memaksa HTTPS
- risiko SIM swap (untuk SMS)

---

## What to test

- expiry OTP dan one-time use
- ketahanan terhadap replay
- rate limiting / lockout
- validasi server-side (hindari “client-side only”)
