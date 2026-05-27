2025-06-24 14:24

Status:

Tags: [[eWPT]] [[SQL Injection]]
###### Prerequisites: 
# Blind SQL Injection

in Blind SQL Injection the attacker inject the  SQL query and The DB excute it but he doesn't see the response and doesn't get any error messages

the attacker typiclly uses techniques to know that the query is executed by using boolean or time delays

### 1. **Boolean-Based Blind SQL Injection (Content-Based)**

In this technique, the attacker injects SQL queries that evaluate to either **TRUE** or **FALSE** and observes how the web application responds. The goal is to infer data by noticing differences in the page content or structure.

- **How it works:**  
    The attacker modifies a parameter in the query and checks if the application’s response changes.
    
- **Example:**
    
    ```sql
    ' OR 1=1 --     → Always true
    ' OR 1=2 --     → Always false
    ```
    
    If the page loads normally in the first case but displays an error or behaves differently in the second, the attacker confirms that the input is being used in a SQL query.
    
- **Usage:**  
    This technique is used to extract data **bit by bit** (e.g., by asking: "Is the first letter of the username 'a'?").
    
![[Pics/Screenshot 2025-06-24 at 2.28.30 PM.png]]

---

### 2. **Time-Based Blind SQL Injection**

When the application shows the same response regardless of the query result, attackers use **time delays** to infer information.

- **How it works:**  
    The attacker sends queries that cause a deliberate delay when a condition is **true**. If the response is slower, the attacker knows the condition is correct.
    
- **Example (MySQL):**
    
 ```sql
   ' OR IF(SUBSTRING(@@version,1,1)='5', SLEEP(5), 0) -- 
```
	
- If the response takes 5 seconds, it means the first character of the database version is '5'.
    
- **Usage:**  
    This is helpful when the application doesn't return error messages or changes in content, making **Boolean-based techniques ineffective**.
    
