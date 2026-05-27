2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prerequisites: [[XML Injection - Overview]]
# XML Tag Injection - Overview

## Definition

**XML Tag Injection** is a type of XML injection where the attacker injects or modifies **XML tags**, changing the structure of the document.

---

## Key characteristics

- focuses on tags more than plain text values
- can disrupt parsing or trigger unexpected logic
- may lead to privilege escalation if the server trusts injected tags

---

## Example (concept)

If an app accepts user-controlled XML and later decides permissions based on tag content, injected tags can change the decision.
