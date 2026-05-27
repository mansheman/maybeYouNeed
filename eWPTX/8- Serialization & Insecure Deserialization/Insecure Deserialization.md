2026-02-06 22:00

Status:

Tags: [[eWPTX]] [[Insecure Deserialization]]
###### Prerequisites: [[What is Deserialization?]]
# Insecure Deserialization

## Overview

**Insecure Deserialization** is a vulnerability that occurs when untrusted or user-controlled data is deserialized by an application without proper validation or security checks.

This can lead to severe consequences such as:

- **Remote Code Execution (RCE)**
- **Privilege Escalation**
- **Data Manipulation**
- **Authentication Bypass**
- **Denial of Service (DoS)**

---

## How It Works

![[Pics/deserialization/attack-flow.svg]]

### The Attack Flow

1. **Serialization**: The application serializes an object
2. **Interception**: Attacker captures or accesses the serialized data
3. **Modification**: Attacker modifies the serialized data to inject malicious payloads
4. **Transmission**: Malicious data is sent to the application
5. **Deserialization**: Application deserializes without validation
6. **Exploitation**: Malicious code executes or unexpected behavior occurs

---

## What Causes the Vulnerability?

### Lack of Input Validation

The application blindly accepts serialized data from untrusted sources without validating its authenticity or integrity.

### Overly Permissive Deserialization Libraries

Many deserialization frameworks automatically:

- Instantiate objects
- Call methods
- Execute code

This increases attack vectors significantly.

### Complex Object Hierarchies

Deserializing data into complex objects or hierarchies can introduce unexpected behaviors, especially if deserialization invokes constructors or methods.

### Lack of Security Controls

Missing cryptographic signing or validation mechanisms allow attackers to tamper with serialized data undetected.

### Trusting User Input

Serialized data from users or external sources is inherently untrusted but is often treated as safe by applications.

---

## Impact

### Remote Code Execution (RCE)

Attackers inject payloads that exploit insecure deserialization processes to execute arbitrary code on the server.

**Example**: Replacing serialized objects with ones containing malicious code.

### Privilege Escalation

Tampering with serialized data to escalate privileges or impersonate users by modifying access tokens or session objects.

### Data Tampering

Changing serialized data values to manipulate application behavior, such as:

- Increasing account balances
- Bypassing payment checks
- Modifying shopping cart contents

### Application Logic Abuse

Injecting objects that invoke unintended application functionality, such as triggering sensitive methods during deserialization.

### Denial of Service (DoS)

Crafting serialized data to exhaust server resources during deserialization, causing application crashes.

---

## Affected Technologies

| Language/Format | Risk Level | Notes |
|-----------------|------------|-------|
| Java Pickle | High | Native deserialization is very powerful |
| Python Pickle | High | Executes arbitrary code by design |
| PHP Serialization | High | Can lead to RCE via object injection |
| .NET BinaryFormatter | High | Dangerous if used on untrusted input |
| Node.js | Medium | Depends on library used |
| JSON | Low | Generally safe, but can be exploited in some contexts |
| XML | Medium | XXE and other XML attacks possible |

---

## Detection Indicators

### Signs of Serialized Data

- Binary data with magic bytes (Java: `ac ed 00 05`)
- Base64-encoded blobs in cookies/tokens
- Data with "O:" prefix (PHP serialization)
- Pickle opcodes in Python data

### Vulnerable Patterns

```python
# DANGEROUS: Deserializing user input without validation
data = pickle.loads(request.body)
```

```java
// DANGEROUS: Deserializing user input
ObjectInputStream in = new ObjectInputStream(request.getInputStream());
Object obj = in.readObject();
```

```php
// DANGEROUS: Unserialize user input
$data = unserialize($_POST['data']);
```

---

## Prevention

### 1. Avoid Deserialization of Untrusted Data

The safest approach is to never deserialize data from untrusted sources.

### 2. Use Safe Data Formats

Prefer simple, safe formats like JSON for data interchange.

```python
# Safer alternative
data = json.loads(request.body)
```

### 3. Implement Integrity Checks

Use cryptographic signatures to verify serialized data hasn't been tampered with.

```python
import hmac
import json

# Sign
signature = hmac.new(key, json.dumps(data).encode(), hashlib.sha256).digest()

# Verify
if hmac.compare_digest(signature, expected_signature):
    data = json.loads(serialized_data)
```

### 4. Use Safe Deserialization Libraries

Use libraries that don't execute arbitrary code during deserialization.

### 5. Restrict Classes

If using Java/Python serialization, restrict which classes can be deserialized.

### 6. Sandbox Isolation

Run deserialization in an isolated environment with restricted permissions.

---

## Key Points

- Insecure deserialization is a critical vulnerability (OWASP A8: 2017, A08: 2021)
- The vulnerability occurs during deserialization of untrusted data
- Language-native serializers (Java, Python Pickle, PHP) are most dangerous
- Prevention: Avoid, validate, use safe formats, implement integrity checks

---

## References

- OWASP Deserialization Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html
- CWE-502: Deserialization of Untrusted Data
- "Marshalling Pickles" (frohoff/beans): https://frohoff.github.io/appseccali-marshalling-pickles/
