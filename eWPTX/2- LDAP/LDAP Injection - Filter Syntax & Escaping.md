2026-02-04 06:27

Status:

Tags: [[eWPTX]] [[LDAP]] [[LDAP Injection]]
###### Prerequisites: [[What is LDAP?]] [[LDAP Injection]]
# LDAP Injection - Filter Syntax & Escaping

## LDAP filters (quick reminder)

LDAP search uses filters like:

- `(&(...)(...))` AND
- `(|(...)(...))` OR
- `(!(...))` NOT

Example:

```text
(&(objectClass=person)(uid=alice))
```

---

## Characters that must be treated as unsafe in filter values

If user input goes into a **filter value**, these characters are critical:

- `*` wildcard
- `(` `)` grouping
- `\\` escape character
- NUL byte

`& | !` are also dangerous when the input can break out of value context.

---

## RFC4515 escaping (filter values)

When you insert a string into a filter value, escape at minimum:

|Character|Escape|
|---|---|
|`*`|`\\2a`|
|`(`|`\\28`|
|`)`|`\\29`|
|`\\`|`\\5c`|
|NUL|`\\00`|

Why it matters:

- Without escaping, input can introduce wildcards or restructure the filter.
- With escaping, special characters become literal characters in the value.

---

## DN escaping is different (RFC4514)

If your app constructs a **DN** (Distinguished Name) from input, DN escaping rules differ from filter escaping.

Examples of DN contexts:

- `cn={name},ou=Users,dc=example,dc=com`

Key point:

- Use the correct escaping function for the correct context (filter vs DN).
