2026-02-04 06:53

Status:

Tags: [[eWPTX]] [[eWPTX/3- ORM/ORM]]
###### Prerequisites: [[What is ORM?]]
# How ORM Works

## 1) Entity-to-table mapping

- Each class (entity/model) corresponds to a database table.
- Each attribute corresponds to a column.

---

## 2) Object-to-row mapping

- An object instance represents a single row.
- The ORM converts rows ↔ objects automatically.

---

## 3) CRUD without writing SQL manually

ORMs expose methods to perform operations:

- Create: `add()` / `save()`
- Read: `get()` / `filter()` / `find()`
- Update: change object then `commit()`
- Delete: `delete()` then `commit()`

The ORM translates these calls into SQL.

---

## 4) Query abstraction

Developers write queries in the ORM’s query language/API.

The ORM generates database-specific SQL behind the scenes.
