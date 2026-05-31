2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prasyarat: [[What is XML?]]
# XML Injection - Gambaran

## Definition

**XML Injection** terjadi ketika penyerang memanipulasi input user untuk menyisipkan konten XML berbahaya ke sistem yang memproses XML.

Ini istilah payung untuk masalah ketika input tak tepercaya mengubah struktur XML atau logika yang digerakkan oleh XML.

---

## Cara kerja (level tinggi)

- Titik masuk: aplikasi menerima XML (API, form, upload file) atau membangun XML dari input user
- Payload: penyerang menyisipkan elemen/atribut/metacharacter
- Dampak: struktur XML berubah → perilaku aplikasi ikut berubah

---

## Key characteristics

- Menarget field input XML atau query berbasis XML
- Bisa menambah/mengubah elemen, atribut, atau data
- Sering dipakai untuk bypass auth, manipulasi data, atau memicu perilaku tak terduga

---

## Example (concept)

Vulnerable XML (app expects one `<user>` structure):

```xml
<user>
  <username>admin</username>
  <password>password123</password>
</user>
```

Jika aplikasi menggabungkan input user ke XML tanpa encoding/validasi yang benar, metacharacter/tag yang disisipkan bisa merusak struktur yang diharapkan.
