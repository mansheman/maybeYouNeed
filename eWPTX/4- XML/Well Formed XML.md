2026-02-04 17:34

Status:

Tags: [[eWPTX]] [[XML]]
###### Prerequisites: [[XML Syntax]]
# Well Formed XML

An XML document with correct syntax is called **Well Formed**.

Key considerations:

- one root element
- tags are case sensitive (`<Letter>` ≠ `<letter>`)
- closing tags are required (no omitted close tags)
- XML prolog is optional, but if present it must come first

---

## Well formed vs valid

- **Well Formed**: syntax rules are correct
- **Valid**: well formed AND validated against a DTD/schema
