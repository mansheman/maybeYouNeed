2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[OAuth]]
###### Prerequisites: [[Introduction to OAuth]]
# Common OAuth Attacks

## Unvalidated `redirect_uri`

If the authorization server does not validate that `redirect_uri` belongs to the client:

- open redirect
- authorization code theft → account takeover

What to test:

- strict redirect URI allow-list
- exact match validation (scheme/host/path)

---

## Weak authorization codes

If authorization codes have low entropy or client secret is weak/not validated, attackers may guess codes.

Test:

- capture multiple codes and analyze randomness/entropy

---

## Everlasting / long-lived authorization codes

If unused codes don’t expire quickly, the window for abuse increases.

Test:

- verify short expiration for codes and one-time use

---

## Codes not bound to client

If a code can be exchanged by a different client, a malicious client can redeem captured codes.

Test:

- ensure code is bound to `client_id` and redirect URI

---

## Weak access/refresh tokens

If tokens are guessable or stored insecurely:

- attackers may brute-force at token endpoint/resource server
- DB compromise reveals usable tokens if stored unhashed

---

## redirect_uri Manipulation (Detailed)

```http
# Normal authorization request
GET /authorize?response_type=code
  &client_id=app123
  &redirect_uri=https://legitimate.com/callback
  &scope=openid profile
  &state=random123

# Attack 1: Open redirect to attacker domain
GET /authorize?response_type=code
  &client_id=app123
  &redirect_uri=https://evil.com/steal
  &scope=openid profile

# Attack 2: Subdomain takeover
&redirect_uri=https://abandoned.legitimate.com/callback

# Attack 3: Path traversal
&redirect_uri=https://legitimate.com/callback/../../../evil-page

# Attack 4: Parameter pollution
&redirect_uri=https://legitimate.com/callback&redirect_uri=https://evil.com
```

---

## State Parameter CSRF

```http
# Normal: state parameter prevents CSRF
GET /authorize?...&state=random_csrf_token

# Attack: remove or reuse state
GET /authorize?response_type=code
  &client_id=app123
  &redirect_uri=https://legitimate.com/callback
  # No state parameter — CSRF possible

# Attacker flow:
# 1. Attacker initiates OAuth, gets code linked to attacker's account
# 2. Crafts URL: https://legitimate.com/callback?code=ATTACKER_CODE
# 3. Victim clicks link → victim's session linked to attacker's account
```

---

## Token Leakage via Referer Header

```http
# Implicit flow returns token in URL fragment
https://legitimate.com/callback#access_token=eyJ...&token_type=bearer

# If callback page loads external resources (images, scripts):
GET /analytics.js HTTP/1.1
Host: third-party.com
Referer: https://legitimate.com/callback#access_token=eyJ...

# The access token leaks via Referer header to third-party
```

---

## Scope Escalation

```http
# Normal request with limited scope
GET /authorize?scope=read:profile

# Attack: request additional scopes
GET /authorize?scope=read:profile write:admin delete:users

# If auth server doesn't validate against registered scopes,
# the token gains elevated permissions
```

---

## Client Secret Leakage

```
# Check these locations for leaked client_secret:
- JavaScript source code (SPA apps)
- Mobile app decompilation (APK/IPA)
- .git/config or .env files exposed
- GitHub/GitLab public repositories
- Browser developer tools (Network tab)

# With client_secret, attacker can:
- Exchange stolen authorization codes
- Request tokens directly (client_credentials flow)
```
