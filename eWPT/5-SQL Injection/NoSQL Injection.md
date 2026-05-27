2025-08-15 17:10

Status:

Tags: [[eWPT]] [[SQL Injection]]
###### Prerequisites: 
## **NoSQL Injection**

### **Definition**

NoSQL Injection is a **security vulnerability** that occurs in applications using **NoSQL databases** (like MongoDB, CouchDB, Cassandra, etc.).  
It allows an attacker to **manipulate queries** by injecting malicious input, leading to:

- Unauthorized access
    
- Data leakage
    
- Unintended operations
    

---

### **How It Works**

- Similar to **SQL Injection**, but instead of SQL commands, the attacker injects **NoSQL query operators** or special syntax.
    
- The vulnerability occurs when **user input is directly embedded** into the query without proper validation or sanitization.
    

---

### **Example Scenario – MongoDB Login**

**Vulnerable Code:**

```javascript
var username = getRequestParameter("username"); // user input
var password = getRequestParameter("password"); // user input

var query = {
    username: username,
    password: password
};

var result = db.users.findOne(query);
if (result) {
    // Login successful
} else {
    // Login failed
}
```

**Normal Input:**

```
username: "john"
password: "1234"
```

→ Checks for `{ username: "john", password: "1234" }`

---

**Malicious Input Example:**

```
username: { $gt: "" }
```

- `$gt` means "greater than."
    
- `{ $gt: "" }` matches **any username greater than an empty string**.
    
- This could cause the query to return the first matching user, bypassing authentication.
    

---

### **Common NoSQL Injection Payloads**

|**Payload**|**Use Case / Effect**|
|---|---|
|`username[$ne]=1&password[$ne]=1`|"Not equals" – Auth bypass|
|`username[$regex]=^adm&password[$ne]=1`|Regex to match usernames starting with "adm"|
|`username[$regex]=.{25}&pass[$ne]=1`|Regex to check value length|
|`username[$eq]=admin&password[$ne]=1`|Equals to "admin"|
|`username[$ne]=admin&pass[$gt]=s`|Greater than check|

---

### **Prevention**

1. **Input Validation & Sanitization** – Reject unexpected types and operators.
    
2. **Use ORMs / ODMs Safely** – Avoid raw query building with user input.
    
3. **Limit Query Operators** – Disable `$gt`, `$ne`, `$regex` from untrusted input.
    
4. **Authentication Controls** – Implement multi-factor authentication.
    
5. **Least Privilege Access** – Restrict DB user permissions.
    

---

🔗 **More Resources:**  
[PayloadsAllTheThings – NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)

---
