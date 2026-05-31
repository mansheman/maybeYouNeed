2026-02-04 17:34

Status:

Tags: [[eWPTX]] [[XML]]
###### Prasyarat: [[Well Formed XML]]
# XML DTD (Document Type Definition)

## What is a DTD?

**DTD** mendefinisikan struktur, elemen, dan atribut dari sebuah dokumen XML.

Fungsinya seperti blueprint yang menentukan bentuk dokumen, sehingga XML bisa dinilai **valid** dan mengikuti aturan yang sudah ditetapkan.

DTD bisa berupa:

- internal (ditanam di dalam XML)
- external (direferensikan dari luar)

---

## DTD building blocks

- **Elements**: dideklarasikan dengan `<!ELEMENT ...>`
- **Attributes**: dideklarasikan dengan `<!ATTLIST ...>`
- **Entities**: dideklarasikan dengan `<!ENTITY ...>`
- `#PCDATA` (parsed character data) dan `CDATA` (raw/unparsed text)

---

## Example (XML with internal DTD)

```xml
<?xml version="1.0"?>
<!DOCTYPE library [
  <!ELEMENT library (book+)>
  <!ELEMENT book (title, author, year, price)>
  <!ELEMENT title (#PCDATA)>
  <!ELEMENT author (#PCDATA)>
  <!ELEMENT year (#PCDATA)>
  <!ELEMENT price (#PCDATA)>
  <!ATTLIST book id ID #REQUIRED>
]>
<library>
  <book id="b1">
    <title>1984</title>
    <author>George Orwell</author>
    <year>1949</year>
    <price>9.99</price>
  </book>
</library>
```
