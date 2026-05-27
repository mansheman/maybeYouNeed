2026-02-04 06:53

Status:

Tags: [[eWPTX]] [[eWPTX/3- ORM/ORM]]
###### Prerequisites: [[SQL Injection]]
# What is ORM?

## Overview

**ORM (Object-Relational Mapping)** is a programming technique used to **map objects in an application** to **tables/rows in a relational database**.

It abstracts database operations, allowing developers to interact with data using **object-oriented code** instead of writing raw SQL queries for every operation.

---

## Bridge between OOP and relational databases

ORM bridges the gap between:

- Object-oriented programming languages (Python, Java, Ruby, PHP, .NET)
- Relational databases (MySQL, PostgreSQL, SQLite, MSSQL)

By mapping:

- **Class** → **Table**
- **Object** → **Row**
- **Attribute/Field** → **Column**

Example mapping:

```text
class User   <->  users table
User.id      <->  users.id
User.email   <->  users.email
```

---

## What ORM gives you (high level)

- a high-level API for CRUD
- automatic transformation (row ↔ object)
- centralized data-access logic
