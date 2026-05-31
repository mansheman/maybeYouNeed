2026-02-04 22:04

Status:

Tags: [[eWPTX]] [[Web Services]]
###### Prasyarat: [[What Are Web Services?]]
# Web Service vs Web Application

## Web Services

- dirancang untuk komunikasi machine-to-machine
- fokus pertukaran data terstruktur
- sering memakai SOAP atau REST
- bukan untuk interaksi manusia secara langsung

## Web Applications

- diakses lewat browser
- dirancang untuk interaksi manusia
- dari situs sederhana sampai aplikasi kompleks (email, medsos, e-commerce)

---

## Tabel perbandingan

|Aspek|Web Services|Web Application|
|---|---|---|
|Tujuan|Fasilitasi pertukaran data antar aplikasi|Memberikan layanan/tugas untuk end-user|
|Interaksi pengguna|Tanpa UI; machine-to-machine|Ada UI untuk manusia|
|Pertukaran data|Data terstruktur|Input user + pemrosesan + tampilan|
|Protokol|SOAP, REST, XML-RPC, JSON-RPC|Umumnya HTTP/HTTPS (via browser)|
|Fokus keamanan|Transmisi + kontrol akses|Auth, authz, validasi, web vuln umum|
|Contoh|Payment gateway API, weather service|Online banking, Amazon, Gmail|
