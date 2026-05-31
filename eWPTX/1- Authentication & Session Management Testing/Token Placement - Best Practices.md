2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]]
###### Prasyarat: [[Token Placement]]
# Penempatan Token - Best Practices

- Utamakan `Authorization: Bearer` untuk API.
- Hindari token di query parameter kecuali benar-benar terpaksa.
- Kalau pakai cookie: set `HttpOnly`, `Secure`, dan `SameSite` yang sesuai.
- Wajib gunakan HTTPS.
- Minimalkan paparan token (hindari lokasi yang gampang kelog/ke-cache).
