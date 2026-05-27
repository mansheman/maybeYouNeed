2025-08-20 13:25

Status:

Tags: [[eWPT]] [[Tags/Command Injection|Command Injection]]
###### Prerequisites: 
# Command Injection

## Overview

- Command injection vulnerabilities occur when an attacker manipulates web application inputs to execute **arbitrary OS commands** on the server.
    
- This is a **critical security risk** that can lead to full compromise of the server.
    

---

## Causes

- **User Input Handling**: Applications accept input via forms, query parameters, etc.
    
- **Lack of Input Sanitization**: Failure to validate, sanitize, or escape user inputs before passing them to system commands.
    
- **Injection Points**: Attackers locate fields or parameters where malicious commands can be inserted.
    

---

## Exploitation

- **Malicious Input**: Attackers use shell metacharacters (`;`, `|`, `` ` ``, `&&`, etc.) to inject commands.
    
- **Command Execution**: The application concatenates attacker input into a shell command.
    
- **Server Execution**: The OS executes the injected command with the web server’s privileges.
    

---

## Impact

- **Unauthorized Execution**: Arbitrary commands run with web server privileges → potential full compromise.
    
- **Data Exfiltration**: Attackers steal files, database content, or system information.
    
- **System Manipulation**: Possible malware installation, privilege escalation, or persistent backdoors.
    

---