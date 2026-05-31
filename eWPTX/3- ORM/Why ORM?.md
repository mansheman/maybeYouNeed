2026-02-04 06:53

Status:

Tags: [[eWPTX]] [[eWPTX/3- ORM/ORM]]
###### Prasyarat: [[What is ORM?]]
# Kenapa ORM?

## Menjembatani object code dan tabel

- Database relasional menyimpan data dalam **tabel**.
- Bahasa OOP merepresentasikan data sebagai **object/class**.

ORM menerjemahkan struktur object menjadi struktur tabel (dan sebaliknya).

---

## Menyederhanakan operasi database (CRUD)

Framework ORM menyediakan API level tinggi untuk operasi umum:

- Create
- Read
- Update
- Delete

Jadi developer bisa fokus ke logika aplikasi, bukan SQL berulang.

---

## Mengurangi boilerplate

ORM mengurangi kerja repetitif seperti:

- mapping row ke object
- menulis query insert/update/select yang sama berulang
- menjaga SQL lebih konsisten di seluruh aplikasi

---

## Lebih “portable” antar database (sampai batas tertentu)

ORM sering mengabstraksi perbedaan sintaks antar database, sehingga porting lebih mudah.

Contoh migrasi:

- SQLite → PostgreSQL
- MySQL → PostgreSQL

---

## Security (kalau dipakai dengan benar)

ORM biasanya mendorong query ter-parameterisasi, yang membantu mitigasi **SQL Injection**.

Catatan penting:

- ORM bukan jaminan aman.
- Salah pakai tetap bisa memunculkan **ORM Injection** atau SQLi klasik lewat raw SQL.
