2025-08-24 09:30

Status:

Tags: [[eWPT]] [[File & Resource Attacks]]
###### Prerequisites: 

# Web Service Implementations

## Overview

- Web service implementations = **different ways web services can be created, deployed, and used**.
    
- Common methods: **SOAP, XML-RPC, JSON-RPC, REST**.
    

---

## 1. XML-RPC (1998)

**Definition:** A simple protocol that lets applications call functions on remote systems using **XML messages over HTTP**.

- Lightweight, precursor to SOAP & REST.
    
- Works by sending **HTTP requests** with method calls in XML.
    

**Request Example**

```xml
<methodCall>
  <methodName>sampleMethod</methodName>
  <params><param><value><int>42</int></value></param></params>
</methodCall>
```

**Response Example**

```xml
<methodResponse>
  <params><param><value><string>Hello, World!</string></value></param></params>
</methodResponse>
```

---

## 2. JSON-RPC

**Definition:** A lightweight protocol that uses **JSON messages** to call functions on remote systems.

- Easier to read and smaller than XML-RPC.
    
- Common in **web apps and microservices**.
    

**Request Example**

```json
{
  "method": "sampleMethod",
  "params": [42],
  "id": 1
}
```

---

## 3. SOAP (Simple Object Access Protocol)

**Definition:** A **heavyweight protocol** using **XML** for exchanging structured data with built-in support for **security, reliability, and transactions**.

- Often paired with **WSDL** for service description.
    
- Considered more formal/strict than REST.
    

**Request Example**

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:web="http://www.example.com/webservice">
  <soapenv:Body>
    <web:sampleMethod>
      <web:inputParameter>42</web:inputParameter>
    </web:sampleMethod>
  </soapenv:Body>
</soapenv:Envelope>
```

**Response Example**

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:web="http://www.example.com/webservice">
  <soapenv:Body>
    <web:sampleMethodResponse>
      <web:result>Hello, World!</web:result>
    </web:sampleMethodResponse>
  </soapenv:Body>
</soapenv:Envelope>
```

---

## 4. REST (Representational State Transfer)

**Definition:** An **architectural style** (not a protocol) that uses **HTTP methods (GET, POST, PUT, DELETE)** to manage resources, often with **JSON**.

- Flexible, scalable, widely used for modern APIs.
    
- Easy to implement and stateless.
    

**Examples**

- `GET http://ws.site/book` → list all books
    
- `GET http://ws.site/book/1` → retrieve a book
    
- `POST http://ws.site/book` → create a book
    
- `PUT http://ws.site/book/1` → update/replace a book
    
- `DELETE http://ws.site/book/1` → delete a book
    

---

📌 **Key Insight**

- **XML-RPC** → Old, simple XML calls.
    
- **JSON-RPC** → Same idea but with JSON.
    
- **SOAP** → Strict, XML-heavy, feature-rich protocol.
    
- **REST** → Flexible, lightweight, HTTP-based style, modern favorite.
    

---
