2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[Session Management]]
###### Prerequisites: [[Testing for Session Management Schema (WSTG-SESS-01)]]
# Cookie Reverse Engineering & Tampering

## Overview

This technique focuses on analyzing and modifying session cookies to identify weaknesses in the application’s session management.

Goal: understand how cookies are structured, encoded, and protected, and whether manipulation could bypass controls.

---

## Reverse engineering (structure & encoding)

What to look for:

- predictable patterns
- encoded data (Base64 / URL encoding)
- user attributes inside cookies (username, userId, role)

If decoding reveals sensitive info and it’s not protected (signed/encrypted), it can enable:

- user enumeration
- privilege manipulation attempts

---

## Cookie tampering (manipulation)

Common tampering ideas (validate server-side behavior):

- change `role=guest` → `role=admin`
- change `userId=1001` → `userId=1002`
- replay a cookie on a different browser/device

Key point:

- server must enforce authorization server-side
- client-side cookie values must not be trusted unless cryptographically protected

---

## What to record in notes

- cookie name(s) and where they appear
- encoding used (if any)
- whether values are signed/encrypted
- server response differences (content/status/redirect)
- security impact (auth/authz/data exposure)

---

## Framework Cookie Formats

### Flask (Python)
```
# Flask session cookie (base64 + signature)
eyJ1c2VyIjoiYWRtaW4ifQ.ZH3...<signature>

# Decode (unsigned part is base64 JSON)
echo "eyJ1c2VyIjoiYWRtaW4ifQ" | base64 -d
# {"user":"admin"}
```

### Laravel (PHP)
```
# Laravel encrypts cookies with APP_KEY
# Cookie value is base64(json(iv + value + mac))
# Decryption requires APP_KEY from .env
```

### ASP.NET
```
# ViewState cookie: serialized with ObjectStateFormatter
# .ASPXAUTH: Forms Authentication ticket
# If machineKey is known, can forge cookies
```

### Express.js (Node)
```
# connect.sid — session ID referencing server-side store
# If using cookie-session: base64 JSON (signed with secret)
```

---

## Tools

### flask-unsign
```bash
# Decode Flask session cookie
flask-unsign --decode --cookie 'eyJ1c2VyIjoiYWRtaW4ifQ.ZH3...'

# Brute-force the secret key
flask-unsign --unsign --cookie 'COOKIE_HERE' --wordlist rockyou.txt

# Forge a new cookie with known secret
flask-unsign --sign --cookie '{"user":"admin","role":"superuser"}' --secret 'found_secret'
```

### Burp Suite Decoder
```
1. Capture cookie in Proxy
2. Send to Decoder tab
3. Apply transforms: URL decode → Base64 decode → view plaintext
4. Modify values → Re-encode → Replace in Repeater
```

### hashid / hash-identifier
```bash
# Identify hash type in cookie value
hashid 'a94a8fe5ccb19ba61c4c0873d391e987982fbbd3'
# [+] SHA-1

# Common cookie hash patterns:
# MD5:    32 hex chars
# SHA1:   40 hex chars
# SHA256: 64 hex chars
```

---

## Cookie Flag Audit

| Flag | What It Does | Test |
|------|-------------|------|
| `Secure` | HTTPS only | Access over HTTP — cookie sent? |
| `HttpOnly` | No JS access | `document.cookie` — visible? |
| `SameSite=Strict` | No cross-site | CSRF form submit — cookie sent? |
| `SameSite=Lax` | GET cross-site OK | POST cross-site — cookie sent? |
| `Path=/admin` | Scope to path | Request to `/` — cookie sent? |
| `Domain=.example.com` | Include subdomains | Subdomain request — cookie sent? |
