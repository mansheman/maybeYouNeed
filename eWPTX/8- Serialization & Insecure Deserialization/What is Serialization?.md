2026-02-06 22:00

Status:

Tags: [[eWPTX]] [[Serialization]]
###### Prerequisites:
# What is Serialization?

## Overview

**Serialization** is the process of converting an object or data structure (e.g., a Python object, a Java object, or a C# object) into a format that can be easily stored or transmitted and then reconstructed (deserialized) later.

The serialized format is often a byte stream, text-based format (e.g., JSON, XML), or a binary representation.

---

## Why Serialize?

Objects are typically serialized to:

- **Persist data** - Save objects to files or databases
- **Transmit data** - Send objects over a network or between processes
- **Cache state** - Store object state for later use
- **Deep copy** - Create independent copies of complex objects

---

## What Are Objects?

In the context of serialization, an **object** refers to a structured data entity in programming that encapsulates:

- **Properties** (data/attributes)
- **Behaviors** (methods/functions)

An object represents a specific instance of a class or data type that can be converted into a storable or transmittable format for later reconstruction.

---

## How Serialization Works

### 1) Converting the Object

The in-memory representation of an object (its properties, methods, and state) is transformed into a specific format.

Example formats include:

- **Binary**: Compact and fast for storage and transmission
- **Text**: Human-readable formats like JSON, XML, or YAML

### 2) Storing or Transmitting

Once serialized, the data can be:

- Stored in a file or database
- Transmitted over a network (via API)
- Passed between processes or components

### 3) Reconstructing (Deserialization)

The serialized data is parsed to recreate the original object or data structure in memory.

The recipient of serialized data should be able to reconstruct (deserialize) the object back to its unchanged original form.

---

## Common Serialization Formats

| Format | Type | Description |
|--------|------|-------------|
| JSON | Text | Human-readable, widely used for APIs |
| XML | Text | Structured markup, self-describing |
| YAML | Text | Human-readable configuration format |
| Pickle | Binary | Python-specific serialization |
| Java Serialization | Binary | Java built-in serialization |
| MessagePack | Binary | Efficient binary JSON replacement |
| Protocol Buffers | Binary | Google's binary format for structured data |
| CBOR | Binary | Concise Binary Object Representation |

---

## Serialization Example (Python)

```python
import pickle
import json

# Example object
data = {"name": "Alice", "age": 30, "roles": ["admin", "user"]}

# Serialize with Pickle (binary)
serialized_pickle = pickle.dumps(data)
print("Pickle Serialized:", serialized_pickle)

# Serialize with JSON (text)
serialized_json = json.dumps(data)
print("JSON Serialized:", serialized_json)
```

---

## Key Points

- Serialization enables data persistence and communication across different systems or components
- The recipient must be able to deserialize the data back to its original form
- Serialized data is typically neither encrypted nor signed by default
- Format choice depends on use case: text (readable) vs binary (compact/fast)

---

## References

- OWASP Deserialization Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html
