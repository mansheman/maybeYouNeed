2026-02-04 06:27

Status:

Tags: [[eWPTX]] [[LDAP]] [[LDAP Injection]]
###### Prerequisites: [[LDAP Injection - Filter Syntax & Escaping]]
# LDAP Injection - Prevention

## 1) Don’t build filters by string concatenation

Bad pattern:

```text
"(uid=" + userInput + ")"
```

Good principles:

- validate input (allow-list)
- escape input for **filter value** context (RFC4515)
- prefer safe APIs that build filters and escape values correctly

---

## 2) Validate with allow-lists

If the field is a username, enforce a strict format.

Example allow-list idea:

- `a-zA-Z0-9._-`

Reject anything containing LDAP metacharacters.

---

## 3) Escape for the correct context

- Filter values: RFC4515 escaping
- DNs: RFC4514 escaping

Don’t mix them.

---

## 4) Reduce blast radius

- Use a least-privileged bind account
- Limit search base DN and scopes
- Return only needed attributes
- Rate-limit endpoints that query the directory
- Log suspicious input patterns and alert

---

## 5) Don’t make auth decisions purely on “search result exists”

Safer authentication patterns:

- verify credentials properly (bind as the user or equivalent secure verification)
- avoid “search user + compare password attribute” designs
