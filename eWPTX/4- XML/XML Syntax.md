2026-02-04 17:34

Status:

Tags: [[eWPTX]] [[XML]]
###### Prasyarat: [[What is XML?]]
# Sintaks XML

## Tag pembuka dan penutup

Setiap elemen punya tag pembuka dan penutup:

```xml
<name>John Doe</name>
```

---

## Atribut

Elemen bisa punya atribut:

```xml
<user id="123">John Doe</user>
```

Satu elemen XML adalah semua yang berada dari start tag sampai end tag.
Satu elemen dapat berisi:

- text
- atribut
- elemen lain
- atau campuran semuanya

---

## Nesting

Elemen bisa di-nest:

```xml
<user>
  <name>John Doe</name>
  <email>johndoe@example.com</email>
</user>
```

---

## Root element

Dokumen XML wajib memiliki **satu root element** yang membungkus semua elemen lain.
