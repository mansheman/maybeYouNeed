2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[2FA]]
###### Prerequisites: [[Two-Factor Authentication (2FA)]]
# 2FA Testing Methodology

## Information gathering

- identify 2FA mechanism (SMS/email/TOTP/app/hardware)
- understand enrollment and recovery
- review config: token length, expiry, rate limiting

---

## Test the authentication flow

- OTP strength (length/entropy)
- OTP expiry (time-based + invalid after use)
- replay attempts (same OTP twice)
- verify transport security (HTTPS)

---

## Rate limiting and lockout

- brute-force resistance (rate limit / temporary lock)
- response differences that leak validity (status/errors)

---

## Advanced areas

- OAuth/OpenID integration bypasses
- token validation location (server-side)
