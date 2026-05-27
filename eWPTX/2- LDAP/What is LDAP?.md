2026-02-04 06:05

Status:

Tags: [[eWPTX]] [[LDAP]]
###### Prerequisites: 
# What is LDAP?

## Overview

**LDAP (Lightweight Directory Access Protocol)** is an application-layer protocol used to **query** and **modify** data in a directory service over **TCP/IP**.

LDAP is the **access protocol**. The **directory service** (e.g., Microsoft Active Directory, OpenLDAP) is what stores the data and defines the schema/structure.

---

## What LDAP is used for

Web apps and enterprise systems commonly use LDAP to:

- **Authenticate users** (validate credentials against a directory)
    
- **Authorize access** (read groups/roles to decide permissions)
    
- **Query directory data** (name, email, department, manager, etc.)
    
- **Manage identities** (create/disable accounts, update attributes)
    

---

## LDAP ports (common)

|Port|Name|Description|
|---|---|---|
|389|LDAP|Default LDAP port (plaintext; often used with StartTLS).|
|636|LDAPS|LDAP over TLS/SSL (encrypted from connection start).|
|3268|Global Catalog|Active Directory Global Catalog over plaintext.|
|3269|Global Catalog (LDAPS)|Active Directory Global Catalog over TLS/SSL.|

---

## LDAP directory structure (DIT)

LDAP data is organized as a hierarchical **Directory Information Tree (DIT)**:

- **Entry**: one object (user, group, device, etc.)
    
- **Attributes**: key/value fields on an entry (e.g., `uid`, `cn`, `mail`)
    
- **DN** (Distinguished Name): the full path to an entry in the tree
    
- **OU** (Organizational Unit): a container for grouping entries
    

Example DIT:

```text
dc=example,dc=com
├── ou=Users
│   ├── cn=John Doe
│   └── cn=Jane Smith
└── ou=Groups
    ├── cn=Admins
    └── cn=Developers
```

Example DN:

`cn=John Doe,ou=Users,dc=example,dc=com`

---

## Object model (schema / objectClass)

LDAP is **object-oriented** in the sense that each entry follows rules defined by its schema (often via `objectClass`).

Example attributes on a user entry:

- `cn` (common name): John Doe
    
- `sn` (surname): Doe
    
- `mail`: john.doe@example.com
    

---

## LDIF format

**LDIF (LDAP Data Interchange Format)** is a standard plain-text format used to represent directory entries and operations (add/modify/delete).

Example LDIF entry:

```ldif
dn: cn=John Doe,ou=Users,dc=example,dc=com
objectClass: inetOrgPerson
cn: John Doe
sn: Doe
mail: john.doe@example.com
```

---

## LDAP filter syntax (queries)

LDAP searches use **filters** with common operators:

- `=` (equal)
    
- `*` (wildcard)
    
- `&` (AND)
    
- `|` (OR)
    
- `!` (NOT)
    

Examples:

- `(cn=John)` → entries where `cn` is exactly `John`
    
- `(cn=J*)` → entries where `cn` starts with `J`
    
- `(|(sn=a*)(cn=b*))` → `sn` starts with `a` OR `cn` starts with `b`
    

---

## How web apps use LDAP (example flow)

- User submits username/password to the web app
    
- The app checks credentials against LDAP (direct bind or service-account search + verify)
    
- The app reads groups/roles from the directory
    
- Access is granted/denied based on that directory data
    

---

## Key points

- LDAP is a **protocol** for accessing directory services, not the storage itself.
    
- The **directory service** defines and enforces the structure (DIT + schema).
    
- The data is hierarchical (tree), not relational tables.
