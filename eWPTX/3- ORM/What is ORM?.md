2026-02-04 06:53

Status:

Tags: [[eWPTX]] [[eWPTX/3- ORM/ORM]]
###### Prasyarat: [[SQL Injection]]
# Apa itu ORM?

## Gambaran singkat

**ORM (Object-Relational Mapping)** adalah teknik pemrograman untuk **memetakan object di aplikasi** ke **tabel/baris pada database relasional**.

ORM mengabstraksi operasi database, jadi developer bisa berinteraksi dengan data lewat **kode berorientasi objek** tanpa harus menulis raw SQL untuk setiap operasi.

---

## Jembatan antara OOP dan database relasional

ORM menjembatani perbedaan antara:

- Bahasa pemrograman OOP (Python, Java, Ruby, PHP, .NET)
- Database relasional (MySQL, PostgreSQL, SQLite, MSSQL)

Dengan pemetaan:

- **Class** → **Table**
- **Object** → **Row**
- **Attribute/Field** → **Column**

Contoh pemetaan:

```text
class User   <->  users table
User.id      <->  users.id
User.email   <->  users.email
```

---

## Apa yang ORM berikan (level tinggi)

- API level tinggi untuk CRUD
- transformasi otomatis (row ↔ object)
- logika akses data lebih terpusat
