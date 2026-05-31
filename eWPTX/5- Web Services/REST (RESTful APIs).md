2026-02-04 22:04

Status:

Tags: [[eWPTX]] [[Web Services]]
###### Prasyarat: [[Web Service Implementations]]
# REST (RESTful APIs)

## Gambaran singkat

REST (Representational State Transfer) adalah gaya arsitektur untuk merancang aplikasi jaringan.

REST bukan protokol; ia kumpulan constraint/prinsip tentang bagaimana API/service memakai HTTP.

REST service sering memakai JSON atau XML, tapi format lain juga bisa.

---

## HTTP method (mapping yang umum)

|Method|Aksi|Contoh|
|---|---|---|
|GET|Ambil resource / list collection|`/book` (list), `/book/1` (view)|
|PUT|Replace / update (atau create jika belum ada)|`/book/1` (replace)|
|POST|Buat resource baru|`/book` (create)|
|DELETE|Hapus resource|`/book/1` (delete)|
