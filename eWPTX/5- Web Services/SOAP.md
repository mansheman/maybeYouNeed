2026-02-04 22:04

Status:

Tags: [[eWPTX]] [[Web Services]] [[XML]]
###### Prerequisites: [[Web Service Implementations]]
# SOAP

## Overview

SOAP (Simple Object Access Protocol) is a protocol for exchanging structured information in web services.

It defines rules for:

- structuring messages (XML)
- defining RPC-style operations
- handling communication over a network (often HTTP)

SOAP often comes with an interface contract via [[WSDL (Web Services Description Language)]].

---

## Request example

```xml
<soapenv:Envelope
  xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
  xmlns:web="http://www.example.com/webservice">
  <soapenv:Header/>
  <soapenv:Body>
    <web:sampleMethod>
      <web:inputParameter>42</web:inputParameter>
    </web:sampleMethod>
  </soapenv:Body>
</soapenv:Envelope>
```

## Response example

```xml
<soapenv:Envelope
  xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
  xmlns:web="http://www.example.com/webservice">
  <soapenv:Header/>
  <soapenv:Body>
    <web:sampleMethodResponse>
      <web:result>Hello, World!</web:result>
    </web:sampleMethodResponse>
  </soapenv:Body>
</soapenv:Envelope>
```
