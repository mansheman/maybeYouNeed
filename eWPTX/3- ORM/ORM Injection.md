2026-02-04 06:54

Status:

Tags: [[eWPTX]] [[eWPTX/3- ORM/ORM]] [[SQL Injection]]
###### Prerequisites: [[SQL Injection Testing]] [[What is ORM?]]
# ORM Injection

## Overview

**ORM Injection** targets applications using an ORM to interact with a database.

It happens when **untrusted input** is used unsafely inside ORM query construction (often by building dynamic query strings), allowing an attacker to influence the underlying SQL.

> ORMs reduce SQLi risk when used correctly, but unsafe patterns can reintroduce injection.

---

## What causes ORM injection

- Improper input validation
- Dynamic query construction (concatenation / string formatting)
- Using raw SQL through ORM APIs with unsanitized input
- Developer misuse (bypassing ORM safeguards)

---

## Vulnerable example (dynamic query)

```python
username = request.args.get("username")
password = request.args.get("password")

# BAD: building query dynamically with user input
user = User.query.filter(
    f"username = '{username}' AND password = '{password}'"
).all()
```

---

## Example impact (authentication bypass)

If an attacker controls `username`, input like:

```text
admin' OR '1'='1
```

can cause the WHERE clause logic to become always-true in some vulnerable constructions.

---

## Safer approach (principles)

- Don’t concatenate strings to build queries.
- Use parameterized ORM APIs.
- Validate inputs with allow-lists where possible.
- Avoid raw SQL unless necessary (and parameterize it).

Also:

- never store plaintext passwords
- use password hashing + secure verification

---

## Framework-Specific Injection

See deep-dive notes:
- [[ORM Injection - Django]] — `raw()`, `extra()`, QuerySet filter injection
- [[ORM Injection - Hibernate]] — HQL injection, JPQL, Criteria API misuse

---

## SQLAlchemy (Python)

```python
# VULNERABLE — string formatting
from sqlalchemy import text
result = db.execute(text(f"SELECT * FROM users WHERE name = '{user_input}'"))

# SAFE — bound parameters
result = db.execute(text("SELECT * FROM users WHERE name = :name"), {"name": user_input})

# SAFE — ORM query
User.query.filter_by(username=user_input).first()

# VULNERABLE — filter with text()
User.query.filter(text(f"username = '{user_input}'")).first()
```

## Eloquent (PHP / Laravel)

```php
// VULNERABLE — raw where
User::whereRaw("username = '$input'")->get();

// SAFE — parameterized raw
User::whereRaw("username = ?", [$input])->get();

// SAFE — Eloquent builder
User::where('username', $input)->first();

// VULNERABLE — dynamic column
$column = $request->input('sort');
User::orderBy($column)->get(); // Attacker controls column name
```

## Entity Framework (.NET)

```csharp
// VULNERABLE — raw SQL interpolation
var users = context.Users
    .FromSqlRaw($"SELECT * FROM Users WHERE Name = '{input}'")
    .ToList();

// SAFE — parameterized
var users = context.Users
    .FromSqlInterpolated($"SELECT * FROM Users WHERE Name = {input}")
    .ToList();

// SAFE — LINQ
var user = context.Users.Where(u => u.Name == input).FirstOrDefault();
```

## Detection Checklist

| Pattern | Risk | Action |
|---------|------|--------|
| String concatenation in queries | Critical | Replace with parameterized |
| `raw()` / `rawQuery()` / `fromSqlRaw()` | High | Audit parameters |
| User input as column/field names | High | Whitelist allowed fields |
| `extra()` / `annotate(RawSQL())` | High | Use ORM builder methods |
| Standard ORM methods (filter_by, Where, etc.) | Low | Generally safe |
