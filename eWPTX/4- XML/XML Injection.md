2026-02-04 17:34

Status:

Tags: [[eWPTX]] [[XML]]
###### Prasyarat: [[What is XML?]]
# XML Injection

## Catatan (MOC)

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

## Gambaran singkat

XML sering dipakai untuk pertukaran data dan konfigurasi, sehingga XML parser dan logika pemrosesan XML berpotensi memunculkan kerentanan model “injection”.

Kategori umum: XML tag injection, XPath injection, XXE, dan pola DoS terhadap parser.

---

## Catatan

- Banyak isu XML itu bukan “bug tag”-nya, tapi cara aplikasi **mem-parse** dan **memercayai** input XML.
- Parser yang salah konfigurasi bisa mengaktifkan perilaku berbahaya (mis. external entity expansion).
