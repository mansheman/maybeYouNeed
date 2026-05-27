2025-06-24 11:54

Status:

Tags: [[eWPT]] [[SQL Injection]]
###### Prerequisites: [[XSS Attacks]]
# Introduction to SQL Injection

**Definition:**  
SQL Injection is a web application vulnerability that allows attackers to inject malicious SQL statements into input fields, manipulating the backend database.

**Cause:**  
Occurs when user input is not properly validated or sanitized, enabling direct execution of SQL queries by the attacker.

**Example Scenario:** 
A login form that directly inserts user input into a SQL query without validation can be exploited. An attacker might enter SQL code in the username field to bypass authentication and gain unauthorized access.

**Impact:**

- Theft of sensitive data
    
- Unauthorized access to user accounts or admin panels
    
- Data manipulation or deletion
    
- Full system compromise in some cases
    

**Affected Systems:**  
Any application or CMS that uses relational databases like:

- MySQL
    
- MSSQL
    
- SQL Server
    
- Oracle
    
- PostgreSQL
    
- And others
    

**Background:**  
Web applications commonly rely on SQL for interacting with databases. SQL is used by developers, administrators, and applications to manage data such as user credentials, content, and system information.![[Pics/Screenshot 2025-06-24 at 1.37.39 PM.png]]