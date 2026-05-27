2025-06-27 18:19

Status:

Tags: [[eWPT]] [[SQL Injection]]
###### Prerequisites: 
---

## Common Injectable Fields

SQL injection vulnerabilities can exist in various input fields within an application. Below are some **commonly injectable fields**:

- **Login Forms**
    
    - Fields like **username** and **password** are common targets.
        
    - If input is not validated or sanitized, attackers can manipulate the SQL query used for authentication.
        
- **Search Boxes**
    
    - Search input fields may be vulnerable if the search query is directly inserted into SQL statements.
        
    - This can allow attackers to access unauthorized data.
        
- **URL Parameters**
    
    - Data passed through URLs (e.g., `?id=1`) can be dangerous if used directly in SQL queries.
        
    - Without validation, these parameters can be exploited for SQL injection.
        
- **Form Fields**
    
    - Any fields in forms—such as **registration**, **contact**, or **comments**—can be targets.
        
    - Vulnerability arises when input is used in SQL without validation or sanitization.
        
- **Hidden Fields**
    
    - Hidden input fields in HTML can also be manipulated.
        
    - If the data is used directly in SQL queries, it can be exploited.
        
- **Cookies**
    
    - Cookies containing user/session data might be included in SQL queries.
        
    - Lack of proper validation can lead to injection vulnerabilities.
        

---
