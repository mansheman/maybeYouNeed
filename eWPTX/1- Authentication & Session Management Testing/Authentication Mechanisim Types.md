2025-10-14 13:36

Status:

Tags:[[eWPTX]] [[Authentication]]
###### Prasyarat:
## Mekanisme Autentikasi

Mekanisme autentikasi adalah metode/proses untuk memverifikasi identitas user atau sistem yang mencoba mengakses aplikasi web atau service.

Tujuannya: memastikan hanya user yang sah yang bisa mengakses resource sensitif.

Di bawah ini ringkasan tipe-tipe mekanisme autentikasi yang umum di web.

---

## Types of Authentication Mechanisms

### Password-Based Authentication

User memberikan username dan password untuk memverifikasi identitas.

---

### Multi-Factor Authentication (MFA)

Combines two or more independent credentials, such as:

- Something you know (password or PIN).
    
- Something you have (smartphone, security token).
    
- Something you are (biometric verification like fingerprint or facial recognition).
    

---

### Two-Factor Authentication (2FA)

A specific type of MFA that requires exactly two factors for authentication, often a password and a one-time code sent via SMS or an authenticator app.

---

### Token-Based Authentication

Menggunakan token (mis. JWT atau OAuth token) yang diterbitkan setelah login dan dipakai untuk request berikutnya, sehingga tidak perlu input kredensial berulang.

---

### Single Sign-On (SSO)

Allows users to log in once and gain access to multiple applications or services without needing to re-enter credentials, often using protocols like SAML or OAuth.

---

### One-Time Passwords (OTP)

Kode sementara yang dikirim via SMS/email untuk satu sesi/aksi tertentu, biasanya dipakai bersama metode autentikasi lain.