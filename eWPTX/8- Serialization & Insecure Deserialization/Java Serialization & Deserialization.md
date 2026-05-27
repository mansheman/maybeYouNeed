2026-02-06 22:00

Status:

Tags: [[eWPTX]] [[Java]] [[Serialization]]
###### Prerequisites: [[What is Serialization?]] [[Insecure Deserialization]]
# Java Serialization & Deserialization

## Overview

In Java, **serialization** is the process of converting an object's state into a byte stream so it can be stored or transmitted.

The serialized data can later be deserialized to recreate the original object in memory.

Java uses the `Serializable` interface to mark objects that can be serialized.

---

## Java Serialization API

Java provides built-in support for serialization and deserialization through:

- `java.io.ObjectOutputStream` - for writing objects (serialization)
- `java.io.ObjectInputStream` - for reading objects (deserialization)

---

## Creating a Serializable Class

To make a class serializable, it must implement the `Serializable` interface:

```java
import java.io.Serializable;

public class Item implements Serializable {

    // serialVersionUID for version control
    private static final long serialVersionUID = 1L;

    int id;
    String name;

    public Item(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

---

## Serializing an Object

```java
import java.io.*;

class Serialize {
    public static void main(String args[]) {
        try {
            // Creating the object
            Item s1 = new Item(123, "book");

            // Creating stream and writing the object
            FileOutputStream fout = new FileOutputStream("data.ser");
            ObjectOutputStream out = new ObjectOutputStream(fout);

            out.writeObject(s1);
            out.flush();

            // Closing the stream
            out.close();

            System.out.println("Serialized data saved to data.ser");

        } catch(Exception e) {
            System.out.println(e);
        }
    }
}
```

---

## Serialized Format

The serialized file (`data.ser`) is in binary format:

```
ac ed 00 05 73 72 00 04 49 74 65 6d  f8 a2 99 5c 7a 67 e8 02
00 02 49 00 02 69 64 4c 00 04 6e  61 6d 65 74 00 12 4c 6a 61
76 61 2f 6c 61 6e 67 2f 53 74 72  69 6e 67 3b 78 70 00 00 00
7b 74 00 04 62 6f 6f 6b
```

### Java Serialization Magic Bytes

The file begins with `ac ed 00 05` - this is the standard Java serialized format signature.

| Bytes | Meaning |
|-------|---------|
| `ac ed` | Magic number ( identifies Java serialization) |
| `00 05` | Version number |

**Wherever you see binary data starting with those bytes, you can suspect that it contains serialized Java objects.**

---

## Deserializing an Object

```java
import java.io.*;

class Deserialize {
    public static void main(String args[]) {
        try {
            // Creating stream to read the object
            ObjectInputStream in = new ObjectInputStream(
                new FileInputStream("data.ser")
            );

            Item s = (Item) in.readObject();

            // Printing the data of the serialized object
            System.out.println(s.id + " " + s.name);

            // Closing the stream
            in.close();

        } catch(Exception e) {
            System.out.println(e);
        }
    }
}
```

Output:

```
123 book
```

The object was properly reconstructed.

---

## Insecure Deserialization Conditions

A potentially exploitable condition in Java occurs when:

1. `readObject()` (or a similar method) is called on user-controlled data
2. A method is later called on that object

### How the Exploit Works

An attacker can craft an object containing multiple nested properties that, when deserialized and called, will execute malicious actions.

This is implemented using:

- **Dynamic Proxy**: Intercepts method calls
- **Invocation Handler**: Defines behavior when methods are called

---

## Transient Fields

Fields marked as `transient` are not serialized:

```java
public class User implements Serializable {
    private String username;
    private transient String password;  // Not serialized!

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }
}
```

---

## Implementing Custom Deserialization

Classes can define custom deserialization behavior using `readObject()`:

```java
private void readObject(ObjectInputStream in)
    throws IOException, ClassNotFoundException {

    in.defaultReadObject();  // Default deserialization

    // Custom validation/logic here
    if (this.id < 0) {
        throw new InvalidObjectException("ID cannot be negative");
    }
}
```

**Warning**: Custom `readObject()` methods can introduce vulnerabilities if not implemented carefully.

---

## Detection in Web Applications

### Signs of Java Serialization

1. **Magic bytes**: `ac ed 00 05` in binary data
2. **Base64-encoded data**: When these bytes are base64-encoded, they start with `rO0`
3. **Headers**: `Content-Type: application/x-java-serialized-object`
4. **File extensions**: `.ser`, `.jss`

### Example: Java serialized object in HTTP

```
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-java-serialized-object
Content-Length: 245

rO0ABXNyABFqYXZhLnV0aWwuSGFzaFNldLpEhZWWuLc0AwAAeHB3CAAAABAAAAAB
...
```

---

## Safe Alternatives

### 1. JSON

```java
import com.fasterxml.jackson.databind.ObjectMapper;

ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(obj);
MyClass obj = mapper.readValue(json, MyClass.class);
```

### 2. XML

```java
import javax.xml.bind.JAXB;

JAXB.marshal(obj, file);
MyClass obj = JAXB.unmarshal(file, MyClass.class);
```

### 3. Use Whitelisting

```java
import java.io.ObjectInputFilter;

class SafeDeserializer {
    static {
        // Create a filter that only allows specific classes
        ObjectInputFilter filter = ObjectInputFilter.Config.createFilter(
            "com.example.SafeClass!*"
        );
        ObjectInputFilter.setFilter(filter);
    }
}
```

---

## Key Points

- Java serialization uses `Serializable` interface and `ObjectOutputStream`/`ObjectInputStream`
- Serialized data starts with magic bytes: `ac ed 00 05`
- `readObject()` on untrusted data can lead to RCE
- Use safer formats (JSON/XML) for external data
- Implement input validation and class whitelisting if serialization is necessary

---

## References

- Java Object Serialization Specification: https://docs.oracle.com/javase/8/docs/platform/serialization/spec/serialTOC.html
- CWE-502: Deserialization of Untrusted Data
- "Marshalling Pickles" - frohoff/beans: https://frohoff.github.io/appseccali-marshalling-pickles/
