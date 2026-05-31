2026-02-04 06:05

Status:

Tags: [[eWPTX]] [[LDAP]]
###### Prasyarat:
# Apa itu LDAP?

## Gambaran singkat

**LDAP (Lightweight Directory Access Protocol)** adalah protokol layer aplikasi untuk **meng-query** dan **memodifikasi** data pada directory service lewat **TCP/IP**.

LDAP adalah **protokol aksesnya**. Sedangkan **directory service** (mis. Microsoft Active Directory, OpenLDAP) adalah sistem yang **menyimpan data** dan **mendefinisikan schema/struktur**.

---

## LDAP dipakai untuk apa?

Aplikasi web dan sistem enterprise umum memakai LDAP untuk:

- **Autentikasi user** (validasi kredensial terhadap directory)
    
- **Otorisasi akses** (membaca group/role untuk memutuskan permission)
    
- **Query data directory** (nama, email, departemen, manager, dll)
    
- **Manajemen identitas** (buat/nonaktifkan akun, update atribut)
    

---

## Port LDAP (umum)

|Port|Nama|Deskripsi|
|---|---|---|
|389|LDAP|Port default LDAP (plaintext; sering dipakai dengan StartTLS).|
|636|LDAPS|LDAP over TLS/SSL (terenkripsi sejak awal koneksi).|
|3268|Global Catalog|Active Directory Global Catalog (plaintext).|
|3269|Global Catalog (LDAPS)|Active Directory Global Catalog (TLS/SSL).|

---

## Struktur directory LDAP (DIT)

Data LDAP disusun hirarkis dalam **Directory Information Tree (DIT)**:

- **Entry**: satu objek (user, group, device, dll)
    
- **Attributes**: field key/value pada entry (mis. `uid`, `cn`, `mail`)
    
- **DN** (Distinguished Name): “path” lengkap ke sebuah entry dalam tree
    
- **OU** (Organizational Unit): kontainer untuk mengelompokkan entry
    

Contoh DIT:

```text
dc=example,dc=com
├── ou=Users
│   ├── cn=John Doe
│   └── cn=Jane Smith
└── ou=Groups
    ├── cn=Admins
    └── cn=Developers
```

Contoh DN:

`cn=John Doe,ou=Users,dc=example,dc=com`

---

## Object model (schema / objectClass)

LDAP bersifat “object-oriented” dalam arti tiap entry mengikuti aturan dari schema (sering via `objectClass`).

Contoh atribut pada entry user:

- `cn` (common name): John Doe
    
- `sn` (surname): Doe
    
- `mail`: john.doe@example.com
    

---

## Format LDIF

**LDIF (LDAP Data Interchange Format)** adalah format plain-text standar untuk merepresentasikan entry directory dan operasi (add/modify/delete).

Contoh entry LDIF:

```ldif
dn: cn=John Doe,ou=Users,dc=example,dc=com
objectClass: inetOrgPerson
cn: John Doe
sn: Doe
mail: john.doe@example.com
```

---

## Sintaks filter LDAP (query)

LDAP search memakai **filter** dengan operator umum berikut:

- `=` (equal)
    
- `*` (wildcard)
    
- `&` (AND)
    
- `|` (OR)
    
- `!` (NOT)
    

Contoh:

- `(cn=John)` → entry di mana `cn` tepat bernilai `John`
    
- `(cn=J*)` → entry di mana `cn` diawali `J`
    
- `(|(sn=a*)(cn=b*))` → `sn` diawali `a` ATAU `cn` diawali `b`
    

---

## Cara web app memakai LDAP (contoh alur)

- User mengirim username/password ke web app
    
- Aplikasi memeriksa kredensial ke LDAP (direct bind atau service-account search + verify)
    
- Aplikasi membaca group/role dari directory
    
- Akses diberikan/ditolak berdasarkan data directory tersebut
    

---

## Poin penting

- LDAP adalah **protokol** untuk mengakses directory service, bukan storage-nya.
    
- **Directory service** yang mendefinisikan dan menegakkan struktur (DIT + schema).
    
- Datanya hirarkis (tree), bukan tabel relasional.
