2026-02-06

Status:

Tags: [[eWPTX]] [[ORM]]
###### Prerequisites: [[ORM Injection]]
# ORM Injection — Hibernate (Java)

## Overview

Hibernate uses HQL (Hibernate Query Language) and the Criteria API. While Criteria API is generally safe, HQL string concatenation leads to injection similar to SQL injection.

---

## HQL Injection

### Vulnerable pattern

```java
// VULNERABLE — string concatenation
String hql = "FROM User WHERE username = '" + userInput + "'";
Query query = session.createQuery(hql);
List results = query.list();

// Input: admin' OR '1'='1
// Query becomes: FROM User WHERE username = 'admin' OR '1'='1'
// Returns all users
```

### Safe pattern

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

## HQL vs SQL Differences

HQL operates on entity objects, not tables. This affects what an attacker can do:

| Feature | SQL | HQL |
|---------|-----|-----|
| Table/column names | Physical | Entity/property names |
| UNION | Supported | Not supported (usually) |
| Subqueries | Full support | Limited |
| Stored procedures | Yes | No |
| `information_schema` | Yes | No |

**Key limitation**: HQL doesn't support `UNION`, so typical UNION-based extraction doesn't work. Attackers rely on boolean-based or error-based techniques.

---

## JPQL Injection

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

## Criteria API (Generally Safe)

```java
// Safe — type-safe query building
CriteriaBuilder cb = session.getCriteriaBuilder();
CriteriaQuery<User> cr = cb.createQuery(User.class);
Root<User> root = cr.from(User.class);
cr.select(root).where(cb.equal(root.get("username"), userInput));
```

The Criteria API parameterizes values internally. However, if column names are user-controlled, it can still be abused:

```java
// VULNERABLE — user controls the field name
String field = request.getParameter("sortBy");
root.get(field);  // Can access any entity property
```

---

## Native SQL Queries

Hibernate allows raw SQL queries, which are fully vulnerable:

```java
// VULNERABLE
String sql = "SELECT * FROM users WHERE username = '" + input + "'";
SQLQuery query = session.createSQLQuery(sql);

// SAFE
SQLQuery query = session.createSQLQuery("SELECT * FROM users WHERE username = :name");
query.setParameter("name", input);
```

---

## Testing for HQL Injection

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

## Mitigation

- Always use parameterized queries (named parameters `:param` or positional `?1`)
- Prefer Criteria API over string-based HQL
- Never concatenate user input into HQL/JPQL/SQL strings
- Validate and whitelist field names if user controls sort/filter fields
- Use input validation as defense-in-depth
