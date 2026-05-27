2025-06-23 13:47

Status:

Tags: [[eWPT]] [[XSS Attacks]]
###### Prerequisites: 

# **Reflected XSS (Non-Persistent)**

**Definition:**  
Reflected XSS is a vulnerability where the malicious script is embedded in a URL and immediately reflected by the web server in the response.

**How It Works:**

- The attacker crafts a URL containing the XSS payload.
    
- The victim is tricked into clicking the link (e.g., via email or a third-party site).
    
- The vulnerable server includes the payload in the response without sanitization.
    
- The victim’s browser executes the script as part of the page content.
    

**Key Point:**  
Unlike stored XSS, the payload is not saved on the server—it is reflected directly in the response.