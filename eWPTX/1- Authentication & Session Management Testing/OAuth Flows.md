2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[OAuth]]
###### Prerequisites: [[Introduction to OAuth]]
# OAuth Flows

In OAuth 2.0, clients obtain access tokens using different flows (grant types). Each flow is designed for a specific application type and trust level. Modern apps typically use **Authorization Code** (often with PKCE); bearer tokens are the default token type.

---

## 1. Authorization Code Grant

The most secure and widely used flow. Intended for **server-side applications** that can securely store a `client_secret`.

### Step 1 — Authorization Request

The client redirects the user's browser to the authorization server:

```
GET /authorize?response_type=code
    &client_id=CLIENT_ID
    &redirect_uri=https://client.example.com/callback
    &scope=openid profile email
    &state=RANDOM_CSRF_STRING
Host: auth.example.com
```

- `state` — a random value the client generates and validates on callback to prevent CSRF.
- `scope` — the permissions being requested.

### Step 2 — Authorization Response

After the user authenticates and consents, the authorization server redirects back:

```
HTTP/1.1 302 Found
Location: https://client.example.com/callback?code=AUTH_CODE&state=RANDOM_CSRF_STRING
```

### Step 3 — Token Exchange (Back-channel)

The client exchanges the authorization code for tokens server-to-server:

```
POST /token HTTP/1.1
Host: auth.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code=AUTH_CODE
&redirect_uri=https://client.example.com/callback
&client_id=CLIENT_ID
&client_secret=CLIENT_SECRET
```

**Response:**
```json
{
  "access_token": "eyJhbGci...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "dGhpcyBpcyBh..."
}
```

### Security Notes

- The authorization code is short-lived and single-use.
- `client_secret` never leaves the server — the browser never sees it.
- Always validate the `state` parameter on callback to prevent CSRF.
- Restrict `redirect_uri` to pre-registered values; open redirects here lead to code theft.

---

## 2. Implicit Grant (DEPRECATED)

Designed for **browser-based (SPA) apps** that cannot securely store a secret. The access token is returned directly in the URL fragment.

```
GET /authorize?response_type=token
    &client_id=CLIENT_ID
    &redirect_uri=https://spa.example.com/callback
    &scope=openid profile
    &state=RANDOM_CSRF_STRING
Host: auth.example.com
```

**Response (redirect):**
```
HTTP/1.1 302 Found
Location: https://spa.example.com/callback#access_token=eyJhbGci...&token_type=Bearer&expires_in=3600&state=RANDOM_CSRF_STRING
```

### Why Implicit Is Deprecated

- **Token in URL fragment** — exposed to browser history, referer headers, and browser extensions.
- **No refresh tokens** — the user must re-authenticate when the access token expires.
- **No back-channel** — there is no secure server-to-server token exchange; everything happens in the browser.
- **Token injection attacks** — an attacker can replace the fragment token in the redirect.

> **Recommendation:** Use Authorization Code with PKCE for SPAs instead. The Implicit grant is removed from the OAuth 2.1 draft specification.

---

## 3. Resource Owner Password Credentials Grant

The user provides their **username and password directly to the client**. Only appropriate when the client is highly trusted (e.g., the authorization server's own first-party app).

```
POST /token HTTP/1.1
Host: auth.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=password
&username=admin@example.com
&password=S3cureP@ss!
&client_id=CLIENT_ID
&client_secret=CLIENT_SECRET
&scope=openid profile
```

### Security Notes

- The client handles raw credentials — this defeats the purpose of OAuth delegation.
- No support for MFA or consent screens at the authorization server level.
- Should **never** be used with third-party clients.
- Removed from the OAuth 2.1 draft specification.

---

## 4. Client Credentials Grant

Used for **machine-to-machine** (M2M) communication where no user context is needed — e.g., a microservice calling another microservice.

```
POST /token HTTP/1.1
Host: auth.example.com
Content-Type: application/x-www-form-urlencoded
Authorization: Basic BASE64(CLIENT_ID:CLIENT_SECRET)

grant_type=client_credentials
&scope=api.read api.write
```

### Security Notes

- The access token represents the **application itself**, not a user.
- No refresh token is issued (the client can simply request a new token).
- Protect the `client_secret` — rotate it regularly and use vault storage.

---

## PKCE Extension (Proof Key for Code Exchange)

PKCE (RFC 7636) mitigates authorization code interception attacks. It is **mandatory** for public clients (SPAs, mobile apps) and recommended for all clients in OAuth 2.1.

### How It Works

1. The client generates a random `code_verifier` (43–128 characters, unreserved URI characters).
2. The client computes the `code_challenge`:
   ```
   code_challenge = BASE64URL(SHA256(code_verifier))
   ```
3. The authorization request includes the challenge:
   ```
   GET /authorize?response_type=code
       &client_id=CLIENT_ID
       &redirect_uri=https://app.example.com/callback
       &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
       &code_challenge_method=S256
       &state=RANDOM_CSRF_STRING
   ```
4. The token request includes the verifier:
   ```
   POST /token
   grant_type=authorization_code
   &code=AUTH_CODE
   &redirect_uri=https://app.example.com/callback
   &client_id=CLIENT_ID
   &code_verifier=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
   ```
5. The server hashes the verifier and compares it to the stored challenge. If they do not match, the request is rejected.

### Why PKCE Matters

- Even if an attacker intercepts the authorization code (e.g., via a malicious app registered to the same custom URI scheme on mobile), they cannot exchange it without the `code_verifier`.
- `code_challenge_method=S256` is strongly preferred over `plain` because `plain` offers no protection if the code is intercepted alongside the verifier.

---

## Flow Selection Summary

| Flow                      | Use Case                      | Status in OAuth 2.1 |
| ------------------------- | ----------------------------- | -------------------- |
| Authorization Code + PKCE | Web apps, SPAs, mobile        | Recommended          |
| Implicit                  | Legacy SPAs                   | **Removed**          |
| Resource Owner Password   | First-party trusted apps only | **Removed**          |
| Client Credentials        | Machine-to-machine            | Supported            |

---

## Diagram

![[Pics/auth/oauth-authorization-code-flow.svg]]

---

## References

- See [[Common OAuth Attacks]] for attack vectors: open redirect abuse, CSRF on callback, token leakage, scope manipulation, and PKCE downgrade attacks.
- RFC 6749 — The OAuth 2.0 Authorization Framework
- RFC 7636 — Proof Key for Code Exchange (PKCE)
- OAuth 2.1 Draft Specification
