2026-02-04 06:27

Status:

Tags: [[eWPTX]] [[LDAP]] [[LDAP Injection]]
###### Prasyarat: [[What is LDAP?]] [[LDAP Injection]]
# LDAP Injection - Filter Syntax & Escaping

## Filter LDAP (pengingat cepat)

LDAP search uses filters like:

- `(&(...)(...))` AND
- `(|(...)(...))` OR
- `(!(...))` NOT

Contoh:

```text
(&(objectClass=person)(uid=alice))
```

---

## Karakter berbahaya dalam konteks *filter value*

Jika input user masuk ke **nilai filter**, karakter berikut kritikal:

- `*` wildcard
- `(` `)` grouping
- `\\` escape character
- NUL byte

`& | !` juga berbahaya jika input bisa “keluar” dari konteks nilai (biasanya karena filter dirakit dengan string concatenation).

---

## Escaping RFC4515 (untuk nilai filter)

Saat memasukkan string ke nilai filter, minimal lakukan escaping berikut:

|Karakter|Escape|
|---|---|
|`*`|`\\2a`|
|`(`|`\\28`|
|`)`|`\\29`|
|`\\`|`\\5c`|
|NUL|`\\00`|

Kenapa penting:

- Tanpa escaping, input bisa menambahkan wildcard atau mengubah struktur filter.
- Dengan escaping, karakter spesial diperlakukan literal sebagai bagian dari value.

---

## Escaping DN itu beda (RFC4514)

Kalau aplikasi membangun **DN** (Distinguished Name) dari input user, aturan escaping untuk DN **berbeda** dari escaping filter.

Contoh konteks DN:

- `cn={name},ou=Users,dc=example,dc=com`

Poin kunci:

- Gunakan fungsi escaping yang benar sesuai konteks (filter vs DN).
