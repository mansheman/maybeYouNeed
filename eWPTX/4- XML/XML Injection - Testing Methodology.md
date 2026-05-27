2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prerequisites: [[XML Injection - Overview]]
# XML Injection - Testing Methodology

## Goal

Confirm whether user input can **break** or **reshape** an XML document and cause meaningful behavior change.

---

## 1) Identify XML entry points

- endpoints that accept `application/xml` or SOAP
- features that import/upload XML
- parameters that get embedded into server-side XML templates

---

## 2) Probe with XML metacharacters

Metacharacters often relevant for XML injection testing:

- `'` and `"` (often affect attribute values)
- `<` and `>` (tag boundaries)
- `&` (entities)

If the app is building XML unsafely, these can trigger:

- parsing exceptions
- different response paths
- different authorization outcomes

---

## 3) Quotes (single/double)

Quotes define attribute values.

If user input is placed inside an attribute and not encoded, quote injection can break the attribute and change the document.

---

## 4) Ampersand (`&`)

`&` begins an entity reference like:

```text
&EntityName;
```

Testing idea:

- inject a fake entity name to see if the parser throws errors
- try malformed entity formats (missing `;`) to see how parsing fails

---

## 5) Angle brackets (`<` `>`)

Angle brackets define tags/comments/CDATA:

```xml
<tagname>value</tagname>
<!-- comment -->
<![CDATA[value]]>
```

Testing idea:

- inject `<` or `>` and check for parse errors
- look for behavior changes that suggest your input ended up in XML context

---

## What to document (like your SQLi notes)

- where XML is accepted/constructed
- expected vs observed behavior
- error messages (if any)
- impact category (auth, authz, data exposure, DoS)
- fix recommendation (contextual encoding + allow-lists + safe parsers)
