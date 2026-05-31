2026-02-05 06:17

Status:

Tags: [[eWPTX]]
###### Prasyarat:
# Melewati Filter & Evasion

## Catatan (MOC)

### Fondasi
[[Regex (Regular Expressions)]]
[[OWASP XSS Filter Evasion Cheat Sheet]]

### Deteksi & Bypass WAF
[[WAF Fingerprinting & Detection]]
[[Encoding & Transformation Bypasses]]
[[WAF Bypass Payloads & Techniques]]
[[Filter Bypass Case Studies]]

## Bagian ini untuk apa?

- bypass filter input secara aman di lab
- memahami apa yang benar-benar diblok filter (bukan asumsi)
- mengenali pola evasion yang umum (encoding/escaping/quirk normalisasi)

> Tetap etis: gunakan hanya untuk pengujian/pelatihan yang berizin.

Intinya: fokusnya bukan “nge-hack”, tapi memahami rantai transformasi input dari client → WAF/proxy → aplikasi → backend.
