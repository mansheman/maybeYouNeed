2026-02-04 17:34

Status:

Tags: [[eWPTX]] [[XML]]
###### Prasyarat: [[What is XML?]]
# Struktur Tree XML

Dokumen XML membentuk **struktur tree** yang dimulai dari **root**, lalu bercabang sampai ke **leaf**.

---

## Example

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bookstore>
  <book category="cooking">
    <title lang="en">Everyday Italian</title>
    <author>Giada De Laurentiis</author>
    <year>2005</year>
    <price>30.00</price>
  </book>
</bookstore>
```

Catatan:

- prolog mendefinisikan versi XML dan encoding
- `<bookstore>` adalah root element
- `<book>` adalah child element
- `<title>`, `<author>`, `<year>`, `<price>` adalah children dari `<book>`
