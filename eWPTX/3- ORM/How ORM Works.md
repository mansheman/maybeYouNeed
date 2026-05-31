2026-02-04 06:53

Status:

Tags: [[eWPTX]] [[eWPTX/3- ORM/ORM]]
###### Prasyarat: [[What is ORM?]]
# Cara Kerja ORM

## 1) Entity-to-table mapping

- Setiap class (entity/model) merepresentasikan sebuah tabel.
- Setiap atribut merepresentasikan sebuah kolom.

---

## 2) Object-to-row mapping

- Satu instance object merepresentasikan satu baris.
- ORM mengubah rows ↔ objects secara otomatis.

---

## 3) CRUD without writing SQL manually

ORM menyediakan method untuk operasi umum:

- Create: `add()` / `save()`
- Read: `get()` / `filter()` / `find()`
- Update: change object then `commit()`
- Delete: `delete()` then `commit()`

ORM akan menerjemahkan pemanggilan method tersebut menjadi SQL.

---

## 4) Query abstraction

Developer menulis query lewat bahasa/API milik ORM.

Di belakang layar, ORM akan menghasilkan SQL sesuai DB yang dipakai.
