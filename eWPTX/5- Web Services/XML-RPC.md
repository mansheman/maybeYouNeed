2026-02-04 22:04

Status:

Tags: [[eWPTX]] [[Web Services]] [[XML]]
###### Prerequisites: [[Web Service Implementations]]
# XML-RPC

## Overview

XML-RPC (Extensible Markup Language - Remote Procedure Call) is a protocol (created in 1998) that uses XML to encode remote procedure calls over HTTP.

Characteristics:

- simple and lightweight
- precursor to SOAP
- typically calls a single remote method per request

---

## Request example (structure)

```xml
<?xml version="1.0"?>
<methodCall>
  <methodName>sampleMethod</methodName>
  <params>
    <param>
      <value><int>42</int></value>
    </param>
  </params>
</methodCall>
```

## Response example (structure)

```xml
<?xml version="1.0"?>
<methodResponse>
  <params>
    <param>
      <value><string>Hello, World!</string></value>
    </param>
  </params>
</methodResponse>
```
