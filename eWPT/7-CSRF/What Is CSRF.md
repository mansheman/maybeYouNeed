2025-08-18 21:08

Status:

Tags: [[eWPT]] [[CSRF]]
###### Prerequisites: 
# What Is CSRF
# Cross-Site Request Forgery (CSRF)

## Definition

- Cross-Site Request Forgery (CSRF) is a web security vulnerability that occurs when an attacker tricks a user into performing actions on a web application without their knowledge or consent.
    
- It exploits the trust a web application places in the user's browser.
    
- In penetration testing, understanding CSRF is essential for identifying and mitigating this risk.
    

## Attack Methodology

- The attacker crafts a malicious request and tricks a user into unknowingly sending it to a vulnerable web application.
    
- Web applications typically trust requests from a user's browser, which CSRF exploits.
    
- Authentication cookies are automatically included with requests, making them a target for exploitation.
    

### Steps

1. Attacker creates a malicious request (e.g., change email, password, or perform a transaction).
    
2. The malicious request is embedded in a web page, email, or another medium.
    
3. Victim loads the content while authenticated in the target application.
    
4. Victim's browser sends the request along with authentication cookies.
    
5. The application processes the request, assuming it is legitimate, leading to account compromise.
    

## Impact

- Unauthorized account modifications (e.g., email, password, profile changes).
    
- Unauthorized financial actions (e.g., fund transfers).
    
- Execution of malicious actions on behalf of the victim.