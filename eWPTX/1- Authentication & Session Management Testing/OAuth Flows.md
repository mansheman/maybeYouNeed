2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[OAuth]]
###### Prasyarat: [[Introduction to OAuth]]
# OAuth Flows

Di OAuth 2.0, client mendapatkan access token lewat beberapa flow (*grant type*) sesuai tipe aplikasi dan level kepercayaan. Aplikasi modern biasanya pakai **Authorization Code** (sering dengan PKCE). Token paling umum adalah **Bearer token**.

---

## 1. Authorization Code Grant

Flow paling aman dan paling sering dipakai. Cocok untuk **server-side application** yang bisa menyimpan `client_secret` dengan aman.

### Step 1 — Authorization Request

Client mengarahkan browser user ke authorization server:

```
GET /authorize?response_type=code
    &client_id=CLIENT_ID
    &redirect_uri=https://client.example.com/callback
    &scope=openid profile email
    &state=RANDOM_CSRF_STRING
Host: auth.example.com
```

- `state` — nilai random dari client yang wajib divalidasi saat callback untuk mencegah CSRF.
- `scope` — izin yang diminta.

### Step 2 — Authorization Response

Setelah user autentikasi dan setuju, authorization server redirect balik:

```
HTTP/1.1 302 Found
Location: https://client.example.com/callback?code=AUTH_CODE&state=RANDOM_CSRF_STRING
```

### Step 3 — Token Exchange (Back-channel)

Client menukar authorization code menjadi token lewat komunikasi server-to-server (*back-channel*):

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

- Authorization code umumnya berumur pendek dan hanya sekali pakai.
- `client_secret` tidak pernah keluar dari server — browser tidak boleh melihatnya.
- Selalu validasi `state` pada callback untuk mencegah CSRF.
- Batasi `redirect_uri` ke allow-list yang sudah didaftarkan; kalau ada open redirect di sini, code bisa dicuri.

---

## 2. Implicit Grant (DEPRECATED)

Awalnya untuk **SPA di browser** yang tidak bisa menyimpan secret. Access token dikembalikan langsung lewat URL fragment.

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

- **Token ada di URL fragment** — mudah terekspos ke history, referer, dan extension.
- **Tidak ada refresh token** — user harus login ulang saat access token expired.
- **Tanpa back-channel** — tidak ada pertukaran token server-to-server yang aman; semuanya di browser.
- **Token injection** — penyerang bisa mengganti token di URL fragment saat redirect.

> Rekomendasi: gunakan Authorization Code + PKCE untuk SPA. Implicit grant sudah dihapus dari draft OAuth 2.1.

---

## 3. Resource Owner Password Credentials Grant

User memasukkan **username/password langsung ke aplikasi client**. Ini hanya masuk akal untuk client yang sangat dipercaya (mis. first-party app milik provider sendiri).

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

- Client memegang kredensial mentah — ini merusak tujuan delegasi OAuth.
- Biasanya tidak mendukung MFA/consent secara proper di sisi authorization server.
- Sebaiknya **jangan** dipakai untuk third-party client.
- Dihapus dari draft OAuth 2.1.

---

## 4. Client Credentials Grant

Dipakai untuk komunikasi **machine-to-machine** (M2M) tanpa konteks user — misalnya microservice memanggil microservice lain.

```
POST /token HTTP/1.1
Host: auth.example.com
Content-Type: application/x-www-form-urlencoded
Authorization: Basic BASE64(CLIENT_ID:CLIENT_SECRET)

grant_type=client_credentials
&scope=api.read api.write
```

### Security Notes

- Access token merepresentasikan **aplikasi**, bukan user.
- Tidak ada refresh token (client tinggal minta token baru saat perlu).
- Lindungi `client_secret` — rotasi berkala dan simpan di vault/secret manager.

---

## PKCE Extension (Proof Key for Code Exchange)

PKCE (RFC 7636) memitigasi serangan *authorization code interception*. PKCE **wajib** untuk public client (SPA, mobile) dan direkomendasikan luas di OAuth 2.1.

### How It Works

1. Client membuat `code_verifier` random (43–128 karakter, karakter URI yang diizinkan).
2. Client menghitung `code_challenge`:
   ```
   code_challenge = BASE64URL(SHA256(code_verifier))
   ```
3. Authorization request menyertakan challenge:
   ```
   GET /authorize?response_type=code
       &client_id=CLIENT_ID
       &redirect_uri=https://app.example.com/callback
       &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
       &code_challenge_method=S256
       &state=RANDOM_CSRF_STRING
   ```
4. Token request menyertakan verifier:
   ```
   POST /token
   grant_type=authorization_code
   &code=AUTH_CODE
   &redirect_uri=https://app.example.com/callback
   &client_id=CLIENT_ID
   &code_verifier=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
   ```
5. Server meng-hash verifier lalu membandingkannya dengan challenge yang tersimpan. Jika tidak cocok, request ditolak.

### Why PKCE Matters

- Walaupun authorization code berhasil dicegat (mis. lewat aplikasi jahat yang mendaftarkan custom URI scheme yang sama di mobile), penyerang tidak bisa menukarnya tanpa `code_verifier`.
- `code_challenge_method=S256` jauh lebih disarankan daripada `plain` karena `plain` tidak memberi proteksi jika code+verifier sama-sama bocor.

---

## Flow Selection Summary

| Flow                      | Use case                      | Status di OAuth 2.1 |
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

- Lihat [[Common OAuth Attacks]] untuk vektor serangan: open redirect abuse, CSRF di callback, kebocoran token, manipulasi scope, dan downgrade PKCE.
- RFC 6749 — The OAuth 2.0 Authorization Framework
- RFC 7636 — Proof Key for Code Exchange (PKCE)
- OAuth 2.1 Draft Specification
