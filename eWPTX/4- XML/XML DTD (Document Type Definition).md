2026-02-04 17:34

Status:

Tags: [[eWPTX]] [[XML]]
###### Prerequisites: [[Well Formed XML]]
# XML DTD (Document Type Definition)

## What is a DTD?

A **DTD** defines the structure, elements, and attributes of an XML document.

It acts like a blueprint that specifies how the document should be structured, ensuring it is **valid** and follows predefined rules.

A DTD can be:

- internal (embedded in the XML)
- external (referenced from outside)

---

## DTD building blocks

- **Elements**: declared with `<!ELEMENT ...>`
- **Attributes**: declared with `<!ATTLIST ...>`
- **Entities**: declared with `<!ENTITY ...>`
- `#PCDATA` (parsed character data) and `CDATA` (raw/unparsed text)

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
