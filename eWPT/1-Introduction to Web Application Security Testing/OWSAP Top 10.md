2025-05-21 19:39

Status:

Tags: [[eWPT]] [[Introduction to Web Application Security Testing]] [[Testing Lifecycle]]
###### Prerequisites:
 [[Web App Pentesting Methodology]] 


# OWASP Top 10 Web Application Security Risks (2021)

- The OWASP Top 10 is a standard document for pen testers, it represents a broad consensus about the most critical security risks to web apps 

1. **Broken Access Control**  
    Improper enforcement of access restrictions allows attackers to gain unauthorized access to systems or data.
    
2. **Cryptographic Failures**  
    Previously termed "Sensitive Data Exposure," this category addresses failures in cryptographic implementations that compromise data confidentiality and integrity.
    
3. **Injection**  
    Flaws such as SQL, NoSQL, and command injection occur when untrusted data is sent to an interpreter, leading to unintended commands or data access.
    
4. **Insecure Design**  
    Introduced in 2021, this category focuses on risks stemming from design flaws and the absence of security controls during the application's design phase.
    
5. **Security Misconfiguration**  
    Involves incorrect or default configurations, open cloud storage, or error messages that reveal sensitive information.
    
6. **Vulnerable and Outdated Components**  
    Using components with known vulnerabilities can compromise application defenses and enable attacks.
    
7. **Identification and Authentication Failures**  
    Previously "Broken Authentication," this category addresses failures in authentication mechanisms, allowing attackers to compromise passwords, keys, or session tokens.
    
8. **Software and Data Integrity Failures**  
    Concerns code and infrastructure that do not protect against integrity violations, such as unsigned software updates or CI/CD pipeline flaws.
    
9. **Security Logging and Monitoring Failures**  
    Lack of proper logging and monitoring can delay the detection of breaches and hinder incident response.
    
10. **Server-Side Request Forgery (SSRF)**  
    Occurs when a server fetches a remote resource without validating the user-supplied URL, potentially leading to internal system access.

# OWASP Top 10 for 2017

- **Injection**  
    Occurs when untrusted data is sent to an interpreter as part of a command or query. This can lead to unintended commands or unauthorized data access.
    
- **Broken Authentication**  
    Involves flaws in authentication and session management, allowing attackers to compromise passwords, keys, or session tokens, or exploit other implementation flaws to assume other users’ identities.
    
- **Sensitive Data Exposure**  
    Many web applications and APIs do not properly protect sensitive data, such as financial, healthcare, and PII. Attackers may steal or modify such weakly protected data.
    
- **XML External Entities (XXE)**  
    Older or poorly configured XML processors evaluate external entity references within XML documents. External entities can be used to disclose internal files, perform internal port scanning, remote code execution, and denial of service attacks.
    
- **Broken Access Control**  
    Restrictions on what authenticated users are allowed to do are often not properly enforced. Attackers can exploit these flaws to access unauthorized functionality and/or data.
    
- **Security Misconfiguration**  
    Security misconfiguration is the most commonly seen issue. This is commonly a result of insecure default configurations, incomplete or ad hoc configurations, open cloud storage, misconfigured HTTP headers, and verbose error messages containing sensitive information.
    
- **Cross-Site Scripting (XSS)**  
    XSS flaws occur whenever an application includes untrusted data in a new web page without proper validation or escaping. XSS allows attackers to execute scripts in the victim’s browser.
    
- **Insecure Deserialization**  
    Insecure deserialization often leads to remote code execution. Even if deserialization flaws do not result in remote code execution, they can be used to perform attacks, including replay attacks, injection attacks, and privilege escalation attacks.
    
- **Using Components with Known Vulnerabilities**  
    Components, such as libraries, frameworks, and other software modules, run with the same privileges as the application. If a vulnerable component is exploited, such an attack can facilitate serious data loss or server takeover.
    
- **Insufficient Logging & Monitoring**  
    Insufficient logging and monitoring, coupled with missing or ineffective integration with incident response, allows attackers to further attack systems, maintain persistence, pivot to more systems, and tamper, extract, or destroy data.


# Usage in Reports and Tests

When documenting a vulnerability, align it to the OWASP ID for clear communication. For example:

> **Finding:** SQL Injection in login parameter  
> **Reference:** OWASP A01:2017 – Injection

This adds standardization and makes the report easier to digest for security engineers and developers.

---

# Notes

- Use the latest OWASP list unless contractually or contextually bound to an older version.  
- OWASP is not exhaustive — use it as a framework, not a complete threat model.  
- Consider integrating OWASP references directly into testing checklists and reporting templates.

Q: 1/2

What is the primary concern related to the OWASP Top 10 vulnerability "Security Misconfiguration"?

Unauthorized access to sensitive files and directories


Q: 2/2

Which of the following OWASP Top 10 vulnerabilities refers to the act of an attacker injecting malicious code into a web application, which is then executed by unsuspecting users?

XSS