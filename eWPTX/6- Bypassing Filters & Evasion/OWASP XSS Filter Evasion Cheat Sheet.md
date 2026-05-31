2026-02-05 06:25

Status:

Tags: [[eWPTX]] [[XSS Attacks]]
###### Prasyarat: [[Bypassing Filters & Evasion]]
# Cheat Sheet OWASP: XSS Filter Evasion

## Referensi

- OWASP XSS Filter Evasion Cheat Sheet: https://owasp.org/www-community/xss-filter-evasion-cheatsheet

> Halaman OWASP ini berisi banyak variasi payload. Gunakan hanya untuk **lab/assessment yang berizin**.

---

## Kenapa ini ada

Banyak aplikasi mencoba mencegah XSS dengan **blacklist** (mis. memblok `<script>`), regex yang lemah, atau encoding parsial.

Biasanya bypass terjadi karena:

- **konteks** yang berbeda (HTML, attribute, JS string, URL, CSS)
- **perilaku parser** yang berbeda (quirk parsing browser, urutan normalisasi/decode)
- **primitive injeksi** yang berbeda (event handler, SVG/XML, URL scheme, dll.)

---

## Takeaway defensif (lebih baik daripada “perang filter”)

- Gunakan **output encoding sesuai konteks** (HTML, attribute, JS, URL, CSS).
- Jika memungkinkan, pakai **allow-list** daripada deny-list.
- Lakukan **normalisasi sekali** (decode/canonicalize), lalu validasi, lalu encode saat output.
- Gunakan framework/template yang auto-escape dan jangan dimatikan.
- Tambahkan **CSP** (Content Security Policy) yang kuat sebagai layer mitigasi.
- Hindari sink berbahaya (`innerHTML`, `document.write`, DOM API yang unsafe) kecuali benar-benar perlu dan aman.

---

## Saat menganalisis “filter” berbasis regex

Kalau filter dibuat sebagai regex, copy lalu uji di regex101 (pilih flavor yang benar) untuk memahami apa yang di-match:

- https://regex101.com

Cari kesalahan umum:

- anchor hilang (`^` / `$`) → hanya match substring
- asumsi case-sensitivity
- decoding tidak konsisten (URL decode / HTML entity / normalisasi Unicode)

