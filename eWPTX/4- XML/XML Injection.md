2026-02-04 17:34

Status:

Tags: [[eWPTX]] [[XML]]
###### Prerequisites: [[What is XML?]]
# XML Injection

## Notes (MOC)

- [[XML Injection - Overview]]
- [[XML Injection - Attack Types]]
- [[XML Injection - Testing Methodology]]
- [[XML Tag Injection]]
- [[XPath Injection]]
- [[XML External Entities (XXE)]]
- [[Billion Laughs Attack]]
- [[XML Bombs]]
- [[XInclude Injection]]
- [[XML Schema Poisoning]]

---

## Overview

XML is heavily used for data exchange and configuration, so XML parsers and XML-processing logic can introduce injection-style vulnerabilities.

Common categories include XML tag injection, XPath injection, XXE, and parser DoS patterns.

---

## Notes

- Many XML issues are about how the app **parses** and **trusts** XML input.
- Misconfigured parsers can enable dangerous behaviors (like external entity expansion).
