2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[2FA]]
###### Prerequisites: [[Two-Factor Authentication (2FA)]]
# Bypassing 2FA

## Common bypass categories (for testing)

### Social engineering

- phishing for password + OTP
- vishing/smishing

### Implementation flaws

- weak recovery flows (security questions, backup codes)
- session fixation around the 2FA boundary
- poor token validation (replay, missing expiry)

### Token interception

- MitM / downgrade issues if not enforcing HTTPS
- SIM swap risks (SMS)

---

## What to test

- OTP expiry and one-time use
- replay resistance
- rate limiting / lockout
- server-side validation (avoid client-side only)
