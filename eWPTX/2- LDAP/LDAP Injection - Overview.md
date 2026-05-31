2026-02-04 06:27

Status:

Tags: [[eWPTX]] [[LDAP]] [[LDAP Injection]]
###### Prasyarat: [[LDAP Injection]] [[What is LDAP?]]
# LDAP Injection - Overview

## Definition

**LDAP Injection** adalah kerentanan web saat **input user disisipkan ke filter/query LDAP** tanpa validasi dan escaping yang benar, sehingga penyerang dapat **memodifikasi logika filter**.

> Inti masalah: input mengubah **struktur query**, bukan sekadar value (konsepnya mirip SQLi, tapi untuk filter LDAP).

---

## Impact

- **Authentication bypass** (mis. aplikasi menganggap “LDAP menemukan user” = login sukses)
    
- **Akses data tanpa izin** (pencarian melebar → lebih banyak entry/atribut)
    
- **Privilege escalation** (jika check group/role berbasis LDAP dan bisa dipengaruhi)
    
- **Information disclosure / user enumeration** (beda hasil atau timing)
    

---

## Where it appears

- Login flow yang melakukan search entry user di LDAP
    
- Halaman directory search (employee/device/printer)
    
- Lookup membership group (mis. “apakah user ada di Admins?”)
    
- Account recovery (cari via email/username lalu ada logic)
    

---

## Why it happens (root causes)

- membangun filter via string concatenation
    
- escaping LDAP tidak ada/salah
    
- validasi lemah (tanpa allow-list untuk username/ID)
    
- memakai hasil LDAP search sebagai keputusan security tanpa guardrail yang kuat
    

---

## Quick mental model

```text
User Input → LDAP filter string → LDAP search results → App decision
```

Intinya: pastikan input user hanya mempengaruhi **value**, bukan **struktur filter**.
