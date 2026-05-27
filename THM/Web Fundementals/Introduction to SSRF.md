2025-07-01 19:32

Status:

Tags: [[THM Web Fundementals]] [[tryhackme]]
###### Prerequisites: 
# What is an SSRF?

**SSRF** stands for **Server-Side Request Forgery**. It is a vulnerability that allows an attacker to make the server perform unauthorized HTTP requests on behalf of the attacker. These requests are usually directed to internal systems or third-party services that the attacker should not be able to access directly.

---

## Types of SSRF

There are two main types of SSRF vulnerabilities:

- **Regular SSRF**  
    The response from the forged request is returned to the attacker, allowing direct data extraction.
    
- **Blind SSRF**  
    The forged request is made, but the response is not visible to the attacker. Detection relies on indirect signals such as DNS logs or timing behavior.
    

---

## What's the Impact?

A successful SSRF attack can lead to:

- Access to unauthorized internal services or endpoints
    
- Exposure of sensitive customer or organizational data
    
- Lateral movement into internal networks
    
- Leakage of authentication tokens or credentials used by the server
    

---



