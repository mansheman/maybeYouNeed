2026-02-04 06:53

Status:

Tags: [[eWPTX]] [[eWPTX/3- ORM/ORM]]
###### Prasyarat: [[What is ORM?]]
# ORM di Aplikasi Web

## Akses database jadi lebih simpel

Framework ORM menyediakan API untuk berinteraksi dengan database lewat object/method.

Ini mengurangi kebutuhan menulis SQL manual untuk flow umum seperti:

- login / manajemen user
- content management
- menyimpan konfigurasi aplikasi
- logging / auditing

---

## Mapping object ke tabel

Contoh:

- `User` class → `users` table
- `username`, `email` → kolom

---

## Benefit yang paling terasa

- development lebih cepat
- boilerplate lebih sedikit
- data model lebih jelas
