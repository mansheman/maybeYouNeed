2026-02-04 22:04

Status:

Tags: [[eWPTX]] [[Web Services]] [[XML]]
###### Prasyarat: [[Web Service Implementations]]
# XML-RPC

## Gambaran singkat

XML-RPC (Extensible Markup Language - Remote Procedure Call) adalah protokol (1998) yang memakai XML untuk meng-encode remote procedure call lewat HTTP.

Karakteristik:

- sederhana dan ringan
- “pendahulu” SOAP
- umumnya memanggil satu method per request

---

## Contoh request (struktur)

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

## Contoh response (struktur)

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
