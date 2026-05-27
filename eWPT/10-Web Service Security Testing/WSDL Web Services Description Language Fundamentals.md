2025-08-24 09:48

Status:

Tags: [[eWPT]] [[File & Resource Attacks]]
###### Prerequisites: 

# WSDL (Web Services Description Language) Fundamentals

## Introduction

- **WSDL = Web Services Description Language** → XML-based language for describing **what a web service does** and **how to use it**.
    
- Acts as a **contract** between provider & consumer.
    
- Commonly used with **SOAP** to define/document SOAP-based services.
    

---

## WSDL Components with Examples

### 1. `<types>`

**Definition:** Defines the data types used by the service (via XML Schema).

```xml
<types>
  <xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema">
    <xsd:element name="GetBookRequest" type="xsd:string"/>
    <xsd:element name="GetBookResponse" type="xsd:string"/>
  </xsd:schema>
</types>
```

➡️ Example: `GetBookRequest` and `GetBookResponse` are defined as strings.

---

### 2. `<message>`

**Definition:** Defines the structure of the input/output messages exchanged.

```xml
<message name="GetBookRequestMessage">
  <part name="bookId" type="xsd:string"/>
</message>

<message name="GetBookResponseMessage">
  <part name="bookTitle" type="xsd:string"/>
</message>
```

➡️ Example: Request contains a `bookId`; Response contains a `bookTitle`.

---

### 3. `<portType>` (WSDL 1.1)

**Definition:** Defines the operations (methods) the service provides.

```xml
<portType name="BookServicePortType">
  <operation name="GetBook">
    <input message="tns:GetBookRequestMessage"/>
    <output message="tns:GetBookResponseMessage"/>
  </operation>
</portType>
```

➡️ Example: Operation `GetBook` uses the request/response messages defined earlier.

---

### 4. `<interface>` (WSDL 2.0)

**Definition:** Replaces `portType`. Defines operations directly referencing schema elements.

```xml
<interface name="BookServiceInterface">
  <operation name="GetBook">
    <input element="tns:GetBookRequest"/>
    <output element="tns:GetBookResponse"/>
  </operation>
</interface>
```

➡️ Example: `GetBook` operation directly references schema-defined elements.

---

### 5. `<binding>`

**Definition:** Specifies the protocol & data format for the operations.

```xml
<binding name="BookServiceSoapBinding" type="tns:BookServicePortType">
  <soap:binding style="document" transport="http://schemas.xmlsoap.org/soap/http"/>
  <operation name="GetBook">
    <soap:operation soapAction="http://example.com/GetBook"/>
    <input><soap:body use="literal"/></input>
    <output><soap:body use="literal"/></output>
  </operation>
</binding>
```

➡️ Example: `GetBook` is bound to SOAP over HTTP with literal XML formatting.

---

### 6. `<service>`

**Definition:** Defines the actual service endpoint (where it can be called).

```xml
<service name="BookService">
  <port name="BookServicePort" binding="tns:BookServiceSoapBinding">
    <soap:address location="http://example.com/BookService"/>
  </port>
</service>
```

➡️ Example: Clients call the service at `http://example.com/BookService`.

---

## Quick Recap

- **Types** → Define data formats.
    
- **Message** → Define input/output message structures.
    
- **PortType / Interface** → Define operations (methods).
    
- **Binding** → Define how operations use protocols.
    
- **Service** → Define endpoint URL.
    

---
