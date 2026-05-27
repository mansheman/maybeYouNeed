2026-02-04 20:43

Status:

Tags: [[eWPTX]] [[XML]]
###### Prerequisites: [[XML Injection]] [[XML DTD (Document Type Definition)]]
# XML External Entities (XXE)

## Overview

The most dangerous type of XML injection often involves injecting **external entities** into the XML document definition. This is known as **XXE (XML eXternal Entities)**.

XXE occurs when an XML parser processes external entities referenced in an XML document in an unsafe way. This can lead to:

- **Data exposure** (reading sensitive server files)
- **SSRF** (accessing internal/external resources from the server)
- **Denial of Service (DoS)** (resource exhaustion via entities/features)

---

## Core idea

In general, the attacker tries to instruct the XML parser to load externally defined entities, making it possible to access sensitive content stored on the host or to make network requests from the server.

---

## External entities: private vs public (concept)

There are two kinds of external entities:

- **Private external entities**: typically restricted in use (single author/group). These are often the most dangerous in exploitation because they can reference local files or internal resources.
- **Public external entities**: designed for broader usage/sharing, usually referencing publicly available identifiers/resources.

Practical takeaway:

- private entity handling is where file disclosure and SSRF commonly come from.

![[Pics/Screenshot 2026-02-04 at 8.44.28 PM.png]]

---

## Technique: resource inclusion (file disclosure)

Scenario: attacker can upload/craft an XML document (or the app builds XML that includes attacker-controlled DTD content), and the parser allows external entity resolution.

### 1) Define an external entity (file)

```xml
<!ENTITY xxefile SYSTEM "file:///etc/passwd">
```

### 2) Reference the entity in the XML body

```xml
<!DOCTYPE message [
  <!ENTITY xxefile SYSTEM "file:///etc/passwd">
]>
<message>
  <body>&xxefile;</body>
</message>
```

### 3) Observe the output channel

To confirm disclosure, you need an application behavior that returns/reflects the parsed value (e.g., error message, rendered field, logs you can read, API response, etc.).

![[Pics/Screenshot 2026-02-04 at 8.44.59 PM.png]]
---

## Notes (MOC)

- [[XXE - Lab (Apache Solr)]]
- [[Blind & OOB XXE]]

## Diagram

![[Pics/xml/xxe-attack-types.svg]]
