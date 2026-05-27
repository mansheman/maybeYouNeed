]2025-06-23 13:42

Status:

Tags: [[eWPT]] [[XSS Attacks]]
###### Prerequisites: [[XSS Types]]

# Stored XSS
**Stored XSS (Persistent)**

**Definition:**  
Stored XSS is a vulnerability where an attacker injects malicious JavaScript code into a web application’s database or source code through an unsanitized input field.

**How It Works:**

- The injected script is stored on the server (e.g., in a database, comment section, or user profile).
    
- When another user visits the affected page, the script is delivered as part of the page content and executed in their browser.
    

**Example Scenario:**  
An attacker submits a malicious script through a comment box. If the input is not sanitized, the script is stored in the database and executed every time a user views the comment.