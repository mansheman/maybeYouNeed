2026-02-04 17:34

Status:

Tags: [[eWPTX]] [[XML]]
###### Prerequisites: [[What is XML?]]
# XML Syntax

## Start and end tags

Every element has an opening and closing tag:

```xml
<name>John Doe</name>
```

---

## Attributes

Elements can have attributes:

```xml
<user id="123">John Doe</user>
```

An XML element is everything from the start tag to the end tag.
An element can contain:

- text
- attributes
- other elements
- or a mix

---

## Nesting

Elements can be nested:

```xml
<user>
  <name>John Doe</name>
  <email>johndoe@example.com</email>
</user>
```

---

## Root element

XML documents must have **one root element** that contains all others.
