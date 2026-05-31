2026-02-06

Status:

Tags: [[eWPTX]] [[ORM]]
###### Prasyarat: [[ORM Injection]]
# Injeksi ORM — Hibernate (Java)

## Gambaran singkat

Hibernate memakai HQL (Hibernate Query Language) dan Criteria API. Criteria API umumnya aman, tetapi HQL yang dirakit via string concatenation bisa memicu injection mirip SQL injection.

---

## HQL injection

### Pola rentan

```java
// VULNERABLE — string concatenation
String hql = "FROM User WHERE username = '" + userInput + "'";
Query query = session.createQuery(hql);
List results = query.list();

// Input: admin' OR '1'='1
// Query becomes: FROM User WHERE username = 'admin' OR '1'='1'
// Returns all users
```

### Pola aman

```java
// SAFE — named parameters
String hql = "FROM User WHERE username = :username";
Query query = session.createQuery(hql);
query.setParameter("username", userInput);
List results = query.list();

// SAFE — positional parameters
String hql = "FROM User WHERE username = ?1";
Query query = session.createQuery(hql);
query.setParameter(1, userInput);
```

---

## Perbedaan HQL vs SQL

HQL bekerja pada entity object, bukan tabel fisik. Ini mempengaruhi kemampuan yang bisa dilakukan lewat injection:

| Fitur | SQL | HQL |
|---------|-----|-----|
| Nama table/column | Fisik | Entity/property |
| UNION | Didukung | Umumnya tidak didukung |
| Subquery | Dukungan penuh | Terbatas |
| Stored procedure | Ya | Tidak |
| `information_schema` | Ya | Tidak |

**Batasan kunci**: HQL tidak mendukung `UNION`, jadi teknik ekstraksi berbasis UNION biasanya tidak jalan. Pola yang sering muncul adalah boolean-based atau error-based.

---

## JPQL injection

JPA (Java Persistence API) uses JPQL, which is similar to HQL:

```java
// VULNERABLE
String jpql = "SELECT u FROM User u WHERE u.email = '" + email + "'";
TypedQuery<User> query = em.createQuery(jpql, User.class);

// SAFE
String jpql = "SELECT u FROM User u WHERE u.email = :email";
TypedQuery<User> query = em.createQuery(jpql, User.class);
query.setParameter("email", email);
```

---

## Criteria API (umumnya aman)

```java
// Safe — type-safe query building
CriteriaBuilder cb = session.getCriteriaBuilder();
CriteriaQuery<User> cr = cb.createQuery(User.class);
Root<User> root = cr.from(User.class);
cr.select(root).where(cb.equal(root.get("username"), userInput));
```

Criteria API melakukan parameterisasi value secara internal. Namun, kalau nama field/property dikontrol user, tetap bisa disalahgunakan:

```java
// VULNERABLE — user controls the field name
String field = request.getParameter("sortBy");
root.get(field);  // Can access any entity property
```

---

## Native SQL queries

Hibernate memungkinkan raw SQL, yang akan rentan jika input disisipkan tanpa parameterisasi:

```java
// VULNERABLE
String sql = "SELECT * FROM users WHERE username = '" + input + "'";
SQLQuery query = session.createSQLQuery(sql);

// SAFE
SQLQuery query = session.createSQLQuery("SELECT * FROM users WHERE username = :name");
query.setParameter("name", input);
```

---

## Pengujian HQL injection

```
# Error-based detection
username=admin'
username=admin' AND '1'='1
username=admin' AND '1'='2

# Boolean-based
username=admin' AND (SELECT COUNT(*) FROM User) > 0 AND '1'='1

# HQL-specific keywords to test
FROM, WHERE, AND, OR, SELECT, ORDER BY, GROUP BY
```

---

## Mitigasi

- Selalu pakai query ter-parameterisasi (named params `:param` atau positional `?1`)
- Lebih baik pakai Criteria API daripada HQL berbasis string
- Jangan pernah concatenate input user ke string HQL/JPQL/SQL
- Validasi + whitelist nama field kalau user mengontrol sort/filter
- Validasi input sebagai defense-in-depth
