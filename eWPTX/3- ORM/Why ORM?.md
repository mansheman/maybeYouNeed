2026-02-04 06:53

Status:

Tags: [[eWPTX]] [[eWPTX/3- ORM/ORM]]
###### Prerequisites: [[What is ORM?]]
# Why ORM?

## Bridge between object code and tables

- Relational databases store data in **tables**.
- OOP languages represent data as **objects/classes**.

ORM translates object structures into table structures and vice versa.

---

## Simplify database operations (CRUD)

ORM frameworks provide a high-level API for common operations:

- Create
- Read
- Update
- Delete

So developers can focus on application logic rather than repetitive SQL.

---

## Reduce boilerplate code

ORM reduces repetitive tasks like:

- mapping rows to objects
- writing repetitive insert/update/select queries
- keeping SQL consistent across the app

---

## Database independence (to some extent)

ORM frameworks often abstract database-specific syntax, making portability easier.

Example migrations:

- SQLite → PostgreSQL
- MySQL → PostgreSQL

---

## Security (when used correctly)

ORMs often encourage parameterized queries, which helps mitigate **SQL Injection**.

Important:

- ORM is not a security guarantee.
- Misuse can still lead to **ORM Injection** or classic SQLi via raw SQL.
