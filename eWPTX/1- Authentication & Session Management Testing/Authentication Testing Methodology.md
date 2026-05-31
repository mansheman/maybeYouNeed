2025-12-29 19:06

Status:

Tags: [[eWPTX]] [[Authentication]]
###### Prasyarat:
# Metodologi Pengujian Autentikasi

## Pengujian Autentikasi

Pengujian autentikasi adalah proses untuk **menguji dan memvalidasi kelemahan** pada mekanisme verifikasi identitas di aplikasi web. Fokusnya bukan cuma “bisa login atau tidak”, tapi apakah penyerang bisa **membypass**, **mengambil alih akun**, atau **menggunakan identitas orang lain**.

Kontrol yang biasanya diuji:

- Form login
- Fitur reset password / lupa password
- Multi-factor authentication (MFA)
- Mekanisme lockout / rate limiting

Jenis kelemahan yang sering dicari:

- Policy password lemah (mudah ditebak/pendek/tanpa kompleksitas)
- Rentan brute force / credential stuffing
- Alur autentikasi yang bisa dibypass (logika/endpoint alternatif)
- Token/cookie ditangani tidak aman
- Session fixation (dibahas di materi session)

### Tujuan

Tujuannya adalah menunjukkan dampak nyata: **akses tidak sah**, **eskalasi privilege**, atau **pembajakan sesi** jika autentikasi gagal melindungi identitas user.

---

## OWASP WSTG (Web Security Testing Guide)

Untuk pentest web, **OWASP WSTG** berguna sebagai checklist/metodologi supaya langkah pengujian autentikasi lebih konsisten dan tidak ada area penting yang terlewat.

Referensi: [OWASP WSTG Project Page](https://owasp.org/www-project-web-security-testing-guide/)

---

## OWASP WSTG - Tes Autentikasi

|Nama Tes|ID|Ringkasannya|
|---|---|---|
|**Kredensial lewat channel terenkripsi**|WSTG-ATHN-01|Pastikan kredensial dikirim via HTTPS (mencegah penyadapan/tampering).|
|**Default credentials**|WSTG-ATHN-02|Cek apakah masih ada akun default (sering jadi pintu masuk cepat).|
|**Lockout mechanism lemah**|WSTG-ATHN-03|Evaluasi lockout/rate limit untuk menahan brute force.|
|**Bypass skema autentikasi**|WSTG-ATHN-04|Cari celah logika/alur yang membuat login bisa dilewati.|
|**"Remember me" rentan**|WSTG-ATHN-05|Pastikan implementasi tidak membocorkan data sensitif atau token yang mudah dicuri/replay.|

---

## Tambahan Tes Autentikasi

|Nama Tes|ID|Ringkasannya|
|---|---|---|
|**Kelemahan browser cache**|WSTG-ATHN-06|Pastikan data sensitif tidak tersimpan di cache (mis. halaman setelah login).|
|**Policy password lemah**|WSTG-ATHN-07|Tinjau kompleksitas, panjang minimal, dan aturan lain yang relevan.|
|**Autentikasi lemah di channel alternatif**|WSTG-ATHN-08|Bandingkan kontrol keamanan web vs API/mobile (sering tidak konsisten).|
