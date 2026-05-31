2026-02-04 06:54

Status:

Tags: [[eWPTX]] [[eWPTX/3- ORM/ORM]] [[SQL Injection]]
###### Prasyarat: [[SQL Injection Testing]] [[What is ORM?]]
# Injeksi ORM

## Gambaran singkat

**ORM Injection** menarget aplikasi yang memakai ORM untuk berinteraksi dengan database.

Ini terjadi saat **input yang tidak tepercaya** dipakai secara tidak aman saat membangun query ORM (sering karena query string dirakit secara dinamis), sehingga penyerang dapat mempengaruhi SQL di belakang layar.

> ORM membantu menurunkan risiko SQLi jika dipakai benar, tapi pola yang tidak aman bisa “menghidupkan lagi” injection.

---

## Penyebab umum ORM injection

- Validasi input yang buruk
- Query dinamis (concatenation / string formatting)
- Pemakaian raw SQL lewat API ORM dengan input tidak disanitasi
- Misuse oleh developer (melewati guardrail ORM)

---

## Contoh rentan (query dinamis)

```python
username = request.args.get("username")
password = request.args.get("password")

# BAD: building query dynamically with user input
user = User.query.filter(
    f"username = '{username}' AND password = '{password}'"
).all()
```

---

## Contoh dampak (bypass autentikasi)

Jika penyerang bisa mengontrol `username`, input seperti:

```text
admin' OR '1'='1
```

dapat membuat logika klausa WHERE menjadi selalu true pada konstruksi yang rentan.

---

## Pendekatan yang lebih aman (prinsip)

- Jangan menggabungkan string untuk membangun query.
- Pakai API ORM yang ter-parameterisasi.
- Validasi input dengan allow-list jika memungkinkan.
- Hindari raw SQL kecuali benar-benar perlu (dan tetap parameterize).

Tambahan:

- jangan simpan password plaintext
- pakai hashing password + verifikasi yang benar

---

## Injection spesifik framework

Lihat catatan pendalaman:
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

## Checklist deteksi

| Pola | Risiko | Aksi |
|---------|------|--------|
| String concatenation di query | Kritis | Ganti dengan parameterized |
| `raw()` / `rawQuery()` / `fromSqlRaw()` | Tinggi | Audit parameter |
| Input user sebagai nama kolom/field | Tinggi | Whitelist field yang boleh |
| `extra()` / `annotate(RawSQL())` | Tinggi | Pakai builder ORM |
| Method ORM standar (filter_by, Where, dll) | Rendah | Umumnya aman |
