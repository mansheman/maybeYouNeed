2026-02-04 22:04

Status:

Tags: [[eWPTX]] [[Web Services]] [[XML]]
###### Prasyarat: [[Web Service Implementations]]
# SOAP

## Gambaran singkat

SOAP (Simple Object Access Protocol) adalah protokol untuk pertukaran informasi terstruktur pada web service.

SOAP mendefinisikan aturan untuk:

- struktur pesan (XML)
- operasi model RPC
- komunikasi via jaringan (seringnya HTTP)

SOAP biasanya punya “kontrak” interface via [[WSDL (Web Services Description Language)]].

---

## Contoh request

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

## Contoh response

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
