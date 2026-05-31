2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[OAuth]]
###### Prasyarat: [[Token-Based Authentication]]
# Pengenalan OAuth

## Gambaran Singkat

OAuth 2.0 adalah standar web utama untuk **otorisasi** antar layanan.

Konsepnya: aplikasi pihak ketiga bisa mengakses resource milik user pada provider, **atas persetujuan user**, tanpa aplikasi itu harus memegang password user.

Catatan penting: OAuth fokus di **authorization/delegation**. Untuk “login” (authentication) biasanya dipasangkan dengan **OpenID Connect (OIDC)**.

---

## Komponen OAuth

- Resource Owner: biasanya end-user
- Client: aplikasi yang meminta akses (relying party)
- Resource Server: API yang menyimpan resource terlindungi
- Authorization Server: mengautentikasi user dan menerbitkan token (sering berperan sebagai IdP)
- User Agent: browser/mobile app

---

## OAuth scopes

Scope merepresentasikan izin/aksi yang diminta, misalnya:

- read
- write
- view_gallery
