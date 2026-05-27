2026-02-04 17:34

Status:

Tags: [[eWPTX]] [[XML]]
###### Prerequisites: [[What is XML?]]
# XML Tree Structure

XML documents form a **tree structure** that starts at the **root** and branches to the **leaves**.

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

Notes:

- prolog defines XML version and encoding
- `<bookstore>` is the root element
- `<book>` is a child element
- `<title>`, `<author>`, `<year>`, `<price>` are children of `<book>`
