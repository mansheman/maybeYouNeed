2026-02-06 22:00

Status:

Tags: [[eWPTX]] [[Serialization]] [[Deserialization]]
###### Prerequisites: [[What is Serialization?]] [[What is Deserialization?]]
# Serialization vs Deserialization

## Overview

**Serialization** and **Deserialization** are inverse operations that enable objects to be converted between in-memory representations and storable/transmittable formats.

---

## Comparison

| Aspect | Serialization | Deserialization |
|--------|---------------|-----------------|
| **Direction** | Object → Format | Format → Object |
| **Purpose** | Store or transmit data | Reconstruct data for use |
| **Input** | In-memory object | Byte stream, JSON, XML, etc. |
| **Output** | Serialized data | Reconstructed object |
| **Use Case** | Persistence, communication | Loading received data |

---

## The Complete Cycle

![[Pics/deserialization/complete-cycle.svg]]

---

## Practical Example

### Serialization (Saving)

```python
import pickle

# Create an object in memory
user_session = {
    "username": "admin",
    "role": "administrator",
    "login_time": "2026-02-06T10:00:00Z",
    "permissions": ["read", "write", "delete"]
}

# Serialize to binary format
serialized = pickle.dumps(user_session)

# Save to file
with open("session.bin", "wb") as f:
    f.write(serialized)
```

### Deserialization (Loading)

```python
import pickle

# Read from file
with open("session.bin", "rb") as f:
    serialized = f.read()

# Deserialize back to object
session = pickle.loads(serialized)

# Use the reconstructed object
print(f"User: {session['username']}")
print(f"Role: {session['role']}")
```

---

## Real-World Use Cases

### 1. Web Sessions

```python
# Server serializes session data
session_data = {"user_id": 123, "csrf_token": "abc123"}
token = serialize(session_data)
set_cookie("session", token)

# Later, server deserializes to validate
session_data = deserialize(get_cookie("session"))
```

### 2. API Communication

```python
# Client sends request
request_obj = {"method": "getUser", "params": [123]}
serialized = json.dumps(request_obj)  # Serialize
send(http_post(serialized))

# Server processes
request_obj = json.loads(http_request.body)  # Deserialize
```

### 3. Database Storage

```python
# Store complex object
article = {"title": "...", "tags": [...], "metadata": {...}}
serialized = pickle.dumps(article)
db.execute("INSERT INTO articles (data) VALUES (?)", (serialized,))

# Retrieve and reconstruct
row = db.execute("SELECT data FROM articles WHERE id = ?", (article_id,))
article = pickle.loads(row[0])
```

---

## Key Takeaways

- **Serialization** prepares data for storage/transmission
- **Deserialization** reconstructs data for application use
- Both operations must use compatible formats
- The security risks primarily exist during deserialization of untrusted data

---

## References

- OWASP Deserialization Cheat Sheet
