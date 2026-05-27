2025-07-25 05:03

Status:

Tags: [[eWPT]] [[SQL Injection]]
###### Prerequisites: 
# Boolean Based SQL Injection

### Overview

Boolean-based SQL injection is a type of **Blind SQL Injection**. It is used when the web application does not return error messages or query results directly but still behaves differently depending on whether a query returns TRUE or FALSE.

This technique allows attackers to **infer information** about the database by observing changes in the application's behavior (e.g., page content, structure, redirects, error vs. success responses).

---

###  How It Works

The attacker injects **boolean logic** (`TRUE` or `FALSE` conditions) into an input field. Depending on how the application processes the SQL query, it may behave differently if the query evaluates to true versus false.

---

### Realistic Example

Consider a vulnerable login form that executes this SQL query:

```sql
SELECT * FROM users WHERE username = '<username>' AND password = '<password>';
```

If an attacker inputs:

- **Username**: `' OR '1'='1`
    
- **Password**: `anything`
    

The query becomes:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = 'anything';
```

This condition will **always return true**, and the attacker may bypass authentication.

---

###  Blind Boolean-Based SQL Injection

Used when the application does **not show query results or errors**. Instead, the attacker uses conditions to **infer** database details.

Example Payload:

```sql
' OR LENGTH(database()) > 5-- 
```

If the page behaves differently when the condition is `TRUE` (e.g., successful login or different layout), the attacker can infer that the database name is longer than 5 characters.

---

###  Key Concepts

|Term|Explanation|
|---|---|
|**Blind SQLi**|No visible errors or data in the response.|
|**Boolean-based**|The payload returns `TRUE` or `FALSE`, and the response behavior changes.|
|**Inference**|Information is extracted by guessing and observing the system's behavior.|

---

### Methodology

#### 1. Identify Injection Points

- Test URL parameters, form inputs, headers, and cookies.
    
- Look for fields where input is used directly in SQL queries.
    

#### 2. Analyze Behavior

- Submit simple payloads like `' OR '1'='1` or `' AND 1=2 --`.
    
- Watch for changes: different page loads, redirects, status codes, or delays.
    

#### 3. Craft Boolean Payloads

Examples:

```sql
' OR 1=1 -- 
' AND 1=2 -- 
' OR LENGTH(database()) > 5-- 
' AND ASCII(SUBSTRING(database(),1,1)) > 100-- 
```

#### 4. Observe Responses

- Page content change?
    
- Redirects or response codes?
    
- Error vs. silent fail?
    
- Time delays (if used with time-based SQLi)?
    

#### 5. Use Binary Search Logic

To extract information like:

```sql
' AND ASCII(SUBSTRING(database(), 1, 1)) > 77 -- 
```

Refine it iteratively (e.g., test 65–90 to get ASCII of first letter).

#### 6. Extract Schema/Data

- **Guess database name**: `LENGTH(database())`, `SUBSTRING(database(), x, y)`
    
- **Find tables**: `FROM information_schema.tables WHERE table_schema=database()`
    
- **Check column existence**: `SELECT column_name FROM information_schema.columns`
    

---

###  Sample Test Payloads

|Goal|Payload|
|---|---|
|Always True|`' OR 1=1 --`|
|Always False|`' AND 1=2 --`|
|Test DB name length|`' AND LENGTH(database())=6 --`|
|Extract DB name (1st char)|`' AND ASCII(SUBSTRING(database(),1,1))=115 --`|
|Check if table exists|`' AND (SELECT COUNT(*) FROM users) > 0 --`|

---

### ✅Tips for Testing

- Use **Burp Suite** or **manual HTTP requests** to test injection.
    
- Use **SQLMap** with `--technique=B` to automate boolean-based attacks.
    
- Always confirm **difference in response** clearly before continuing.
    
- Combine with **time-based SQLi** if no visible difference is observed.
    

---

in this case when 1=1 it looged us into the staff account
![[Pics/Screenshot 2025-07-25 at 5.17.39 AM.png]]

but when we type 1=2 it shows an error that is not related to SQL

![[Pics/Screenshot 2025-07-25 at 5.18.08 AM.png]]![[Pics/Screenshot 2025-07-25 at 5.18.28 AM.png]]