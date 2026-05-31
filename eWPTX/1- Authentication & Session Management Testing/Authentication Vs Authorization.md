2025-10-14 13:23

Status:

Tags:[[eWPTX]] [[Authentication]]
###### Prasyarat: [[HTTP]] [[Introduction to Web Application Security Testing]] [[Web Proxies and Web Information Gathering]]
# Autentikasi

Autentikasi adalah proses untuk **memverifikasi identitas** user atau sistem sebelum diberi akses ke resource. Di web app, hasil autentikasi biasanya berupa **session** (cookie) atau **token** (mis. JWT) yang menjadi “bukti” bahwa user sudah login.

Kalimat gampangnya: autentikasi menjawab pertanyaan **“siapa kamu?”**

---

## Autentikasi vs Otorisasi

Otorisasi adalah proses untuk menentukan **aksi apa yang boleh dilakukan** oleh user yang sudah terautentikasi (akses endpoint tertentu, role admin, izin edit, dsb).

Kalimat gampangnya: otorisasi menjawab **“kamu boleh ngapain?”**

|Autentikasi|Otorisasi|
|---|---|
|**Definisi**|Memverifikasi identitas user/sistem.|Menentukan izin/akses setelah identitas tervalidasi.|
|**Tujuan**|Menetapkan “siapa” user tersebut.|Menetapkan “boleh apa” user tersebut.|
|**Proses**|Validasi kredensial (password, OTP, token, sertifikat).|Evaluasi policy/role/permission untuk sebuah aksi/resource.|
|**Hasil**|User dianggap login/terverifikasi (session/token aktif).|Aksi diizinkan atau ditolak (mis. 403).|
|**Contoh**|Login dengan username + password.|User biasa tidak boleh akses `/admin`.|

---

## Kenapa Autentikasi Penting

- Ini “gerbang pertama” sebelum kontrol lain bekerja.
- Kelemahan autentikasi sering berujung ke **account takeover** (ATO): brute force, credential stuffing, reset password yang lemah, dll.
- Kalau autentikasi jebol, kontrol otorisasi juga sering ikut terdampak (privilege escalation/hijack session).

---

## Jenis Autentikasi (ringkas)

- **Knowledge factor**: yang user tahu (password, PIN).
- **Possession factor**: yang user punya (OTP, authenticator app, hardware token).
- **Inherence factor**: yang melekat pada user (biometrik).
- **Federated/SSO**: login via identity provider (OAuth/OIDC, SAML).
- **Certificate-based**: client certificate / mTLS (umum di enterprise).