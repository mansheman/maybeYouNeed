2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[2FA]]
###### Prerequisites: [[OTP Security]]
# OTP Rate Limiting

OTP rate limiting prevents brute force and abuse by restricting the number of verification attempts in a time window.

What to check:

- per-user and per-IP throttling
- progressive delays
- temporary lockout after N failures
- monitoring/alerting
