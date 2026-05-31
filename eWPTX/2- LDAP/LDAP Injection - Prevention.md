2026-02-04 06:27

Status:

Tags: [[eWPTX]] [[LDAP]] [[LDAP Injection]]
###### Prasyarat: [[LDAP Injection - Filter Syntax & Escaping]]
# LDAP Injection - Prevention

## 1) Jangan merakit filter via string concatenation

Pola buruk:

```text
"(uid=" + userInput + ")"
```

Prinsip yang benar:

- validasi input (allow-list)
- escape input untuk konteks **nilai filter** (RFC4515)
- prioritaskan API yang aman (builder) yang menangani escaping dengan benar

---

## 2) Validasi dengan allow-list

Kalau field-nya username, paksa format yang ketat.

Contoh ide allow-list:

- `a-zA-Z0-9._-`

Tolak apa pun yang mengandung metacharacter LDAP.

---

## 3) Escape sesuai konteks yang benar

- Filter values: RFC4515 escaping
- DNs: RFC4514 escaping

Jangan tertukar.

---

## 4) Kecilkan blast radius

- Pakai bind account dengan hak minimal (least privilege)
- Batasi base DN dan scope pencarian
- Kembalikan hanya atribut yang dibutuhkan
- Rate-limit endpoint yang melakukan query directory
- Log pola input mencurigakan dan buat alert

---

## 5) Jangan ambil keputusan auth hanya dari “search result exists”

Pola autentikasi yang lebih aman:

- verifikasi kredensial dengan benar (bind sebagai user atau mekanisme verifikasi yang setara)
- hindari desain “search user + compare password attribute”
