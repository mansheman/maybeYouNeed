2026-02-06 22:00

Status:

Tags: [[eWPTX]] [[Deserialization]]
###### Prerequisites: [[What is Serialization?]]
# What is Deserialization?

## Overview

**Deserialization** is the process of converting serialized data (e.g., a byte stream, JSON string, or binary format) back into its original in-memory representation, such as an object or data structure.

This process reconstructs the data's structure and state, enabling operations or interactions as if the data had never been serialized.

---

## The Serialization/Deserialization Flow

![[Pics/deserialization/serialization-flow.svg]]

---

## How Deserialization Works

### 1) Retrieve Serialized Data

The serialized data is retrieved from:

- Storage (file, database, cache)
- Network (API response, message queue)
- Memory (inter-process communication)

### 2) Parse and Reconstruct

The deserialization process:

- Reads the serialized format
- Parses the structure and data types
- Recreates objects in memory
- Restores the original state

### 3) Use the Reconstructed Object

Once deserialized, the object can be used:

- Access its properties and methods
- Modify its state
- Pass it to other functions

---

## Deserialization Example (Python)

```python
import pickle

# Serialized data (from previous example)
serialized_data = b'\x80\x04\x95\x1e\x00\x00\x00\x00\x00\x00\x00}\x94(\x8c\x04name\x94\x8c\x05Alice\x94\x8c\x03age\x94K\x1e\x8c\x05roles\x94]\x94(\x8c\x05admin\x94\x8c\x04user\x94e.'

# Deserialize the object
deserialized_data = pickle.loads(serialized_data)
print("Deserialized Data:", deserialized_data)
print("Name:", deserialized_data["name"])
print("Age:", deserialized_data["age"])
```

Output:

```
Deserialized Data: {'name': 'Alice', 'age': 30, 'roles': ['admin', 'user']}
Name: Alice
Age: 30
```

---

## Popular Serialization Formats

The most popular and widely used data format for serialization is **JSON**. Prior to JSON, **XML** was the dominant format.

Many programming languages provide built-in capabilities for object serialization, often offering more features than formats like JSON or XML, including:

- Greater flexibility
- Customization of the serialization process
- Support for complex object graphs
- Preservation of type information

---

## Security Considerations

> **Important**: Serialized data is typically **neither encrypted nor signed** by default. This means that, once intercepted, it can often be freely tampered with, potentially leading to unexpected or malicious behavior in the component responsible for deserialization.

In some cases, transport protocols may incorporate serialization alongside compression or encryption, which can conceal or secure the serialized data. However, serialized data may also be encountered in an unprotected, plaintext format.

This scenario is particularly significant for penetration testers and attackers, as it provides an opportunity to:

- Analyze the serialized data structure
- Modify the data to inject malicious payloads
- Bypass security controls through object manipulation

---

## Key Points

- Deserialization reverses serialization, reconstructing objects from stored/transmitted data
- The format must match between serialization and deserialization
- Built-in language serializers (like Pickle, Java serialization) can be dangerous if used on untrusted data
- Always validate and sanitize serialized data from untrusted sources

---

## References

- CWE-502: Deserialization of Untrusted Data
- OWASP Top 10 2021: A8 - Software and Data Integrity Failures
