2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prerequisites: [[What is XML?]]
# XML Injection - Overview

## Definition

**XML Injection** occurs when an attacker manipulates user input to inject malicious XML content into a system that processes XML.

It’s a broad term covering issues where untrusted input changes XML structure or XML-driven logic.

---

## How XML injection works (high level)

- Input point: app accepts XML (API, form, file upload) or builds XML from user input
- Payload: attacker injects elements/attributes/metacharacters
- Impact: XML structure changes → app behavior changes

---

## Key characteristics

- Targets XML input fields or XML queries
- Can inject new elements, attributes, or data
- Often used to bypass authentication, manipulate data, or trigger unexpected behavior

---

## Example (concept)

Vulnerable XML (app expects one `<user>` structure):

```xml
<user>
  <username>admin</username>
  <password>password123</password>
</user>
```

If the app concatenates user input into XML without proper encoding/validation, injected metacharacters/tags can break the intended structure.
