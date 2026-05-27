2025-06-27 18:24

Status:

Tags: [[eWPT]] [[SQL Injection]]
###### Prerequisites: 
# Finding SQL Vulnerabilites (Theoritical)

Identifying SQL injection vulnerabilities involves both **manual testing** and **automated scanning**.

---

###  Manual Testing

- **Malicious Input Testing**
    
    - Inject SQL statements or special characters into input fields (e.g., login forms, search boxes, URL parameters).
        
    - Look for unexpected behavior or SQL-related error messages.
        
- **Error-Based Testing**
    
    - Submit malformed input to trigger SQL errors.
        
    - Useful for revealing database errors or the structure of SQL queries.
        
- **Union-Based Testing**
    
    - Inject `UNION SELECT` queries to retrieve data from other tables.
        
    - If successful, the app may be vulnerable to SQL injection.
        
- **Boolean-Based Testing**
    
    - Test with conditions like `' OR '1'='1` to alter the logic of queries.
        
    - If the response changes, the input is likely being interpreted as SQL.
        
- **Time-Based Testing**
    
    - Use time delays (e.g., `SLEEP(5)`) to detect blind SQL injection.
        
    - If the server response is delayed, the vulnerability may exist.
        
- **Code Review: Input Validation & Sanitization**
    
    - Look at how user input is handled in the code.
        
    - Red flags: user input is **concatenated into SQL queries** without proper sanitization or use of **prepared statements**.
        

---

###  Automated Testing

- **Automated Scanners**
    
    - Tools that detect and exploit SQLi vulnerabilities:
        
        - `SQLMap`
            
        - `OWASP ZAP`
            
        - `Burp Suite`
            
    - These tools simulate attacks and analyze server responses to identify flaws.
        

---
