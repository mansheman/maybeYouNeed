2025-06-24 14:08

Status:

Tags: [[eWPT]] [[SQL Injection]]
###### Prerequisites: 
# In-Band SQLI

its when the hacker injects the sql query into a channel in the web application and recives the result in the same channel
Its dangours because when u try to dump the data base it will return to you in the same page that means you don't have to use any tools like SQL MAP
![[Pics/Screenshot 2025-06-24 at 2.08.05 PM.png]]

### Types of In-Band SQLI

-  **Error-based SQLI**: when attacker injects sql malicious code that brings up errors that may contaion important information about the databes like schemas or content of database itself which the attacker use to exploit further 
	
	
	- **Example (MySQL):**

```sql
`' AND 1=CONVERT(int, (SELECT TOP 1 name FROM users)) --` 
```
If the database throws an error like:  
```sql
"Conversion failed when converting the varchar value 'admin' to data type int"
```

— the attacker now knows that the first user is `'admin'`.

- ![[Pics/Screenshot 2025-06-24 at 2.17.14 PM.png]]

- **Uinon-based SQLI** : its when the attacker combine two queries to inject like the webapp query and his query and the database excute also the second query that the attacker injected and he's doing that using the union operator
- Here’s the same format for the two **In-Band SQL Injection** types, with clear explanations and practical examples:

- **Example (MySQL):**

```sql
 ' UNION SELECT null, username, password FROM users -- 
```
If the original query expected 3 columns, this query will inject user data into the application’s normal response.  
Result on the page might show:

```sql
    null | admin | 12345
```
- **Usage:**  
    Ideal when the application displays query results directly on the page and the attacker knows or can guess the number of columns.

