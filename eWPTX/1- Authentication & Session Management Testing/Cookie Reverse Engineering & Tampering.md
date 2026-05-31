2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[Session Management]]
###### Prasyarat: [[Testing for Session Management Schema (WSTG-SESS-01)]]
# Cookie Reverse Engineering & Tampering

## Gambaran Singkat

Teknik ini fokus pada analisis dan modifikasi **session cookie** untuk mencari kelemahan pada manajemen sesi.

Goal-nya: pahami bagaimana cookie disusun, di-encode, dan diproteksi (signed/encrypted), serta apakah perubahan nilai bisa membypass kontrol.

---

## Reverse engineering (struktur & encoding)

Yang dicari:

- pola yang “terlalu rapih”/mudah ditebak
- data yang di-encode (Base64 / URL encoding)
- atribut user di dalam cookie (username, userId, role)

Kalau hasil decode mengungkap data sensitif dan tidak ada proteksi (signature/encryption), ini bisa membuka jalan ke:

- enumerasi user
- percobaan manipulasi privilege

---

## Cookie tampering (manipulasi)

Ide manipulasi umum (tujuannya menguji perilaku server-side):

- ubah `role=guest` → `role=admin`
- ubah `userId=1001` → `userId=1002`
- replay cookie di browser/device lain

Poin kunci:

- otorisasi harus dipaksa di server-side
- nilai cookie dari client tidak boleh dipercaya kecuali benar-benar diproteksi secara kriptografis

---

## Apa yang dicatat di notes

- nama cookie dan muncul di mana (request/response mana)
- encoding yang dipakai (kalau ada)
- apakah value ditandatangani/terenkripsi
- perbedaan response server (konten/status/redirect)
- dampak keamanan (auth/authz/kebocoran data)

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
1. Tangkap cookie di Proxy
2. Kirim ke tab Decoder
3. Apply transform: URL decode → Base64 decode → lihat plaintext
4. Ubah value → re-encode → replace di Repeater
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

| Flag | Fungsi | Cara uji |
|------|-------------|------|
| `Secure` | HTTPS only | Akses via HTTP — cookie ikut terkirim? |
| `HttpOnly` | Tidak bisa diakses JS | `document.cookie` — terlihat? |
| `SameSite=Strict` | Tidak cross-site | CSRF form submit — cookie ikut terkirim? |
| `SameSite=Lax` | GET cross-site bisa | POST cross-site — cookie ikut terkirim? |
| `Path=/admin` | Scope dibatasi path | Request ke `/` — cookie ikut terkirim? |
| `Domain=.example.com` | Termasuk subdomain | Request dari subdomain — cookie ikut terkirim? |
