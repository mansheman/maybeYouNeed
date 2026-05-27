2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[2FA]]
###### Prerequisites: [[Two-Factor Authentication (2FA)]]
# OTP Security

## Overview

OTPs are temporary, single-use codes used to verify identity during login/transactions.

Key security properties:

- short-lived
- single-use
- rate-limited

---

## OTP methods

- TOTP (time-based)
- SMS OTP
- Email OTP

---

## Good controls

- strict expiry window
- invalidation after use
- rate limiting and lockout
- secure delivery (HTTPS, avoid downgrade)
