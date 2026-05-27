2026-02-04 06:27

Status:

Tags: [[eWPTX]] [[LDAP]] [[LDAP Injection]]
###### Prerequisites: [[LDAP Injection]] [[What is LDAP?]]
# LDAP Injection - Overview

## Definition

**LDAP Injection** is a web vulnerability where **user input is embedded into an LDAP filter/query** without proper validation and escaping, allowing the attacker to **modify the filter logic**.

> The core problem: input changes **query structure**, not just a value (same idea as SQLi, but for LDAP filters).

---

## Impact

- **Authentication bypass** (if the app treats “LDAP found a user” as successful auth)
    
- **Unauthorized data access** (broadened searches → more entries/attributes)
    
- **Privilege escalation** (if group/role checks are LDAP-backed and can be influenced)
    
- **Information disclosure / user enumeration** (different results or timing)
    

---

## Where it appears

- Login flows that search LDAP for the user entry
    
- Directory search pages (employees, devices, printers)
    
- Group membership lookups (e.g., “is user in Admins?”)
    
- Account recovery flows (search by email/username, then apply logic)
    

---

## Why it happens (root causes)

- building filters with string concatenation
    
- missing/incorrect LDAP escaping
    
- weak validation (no allow-list for usernames / IDs)
    
- using LDAP search results as a security decision without hard guarantees
    

---

## Quick mental model

```text
User Input → LDAP filter string → LDAP search results → App decision
```

Fix is to ensure user input can only affect the **value**, never the **filter structure**.
