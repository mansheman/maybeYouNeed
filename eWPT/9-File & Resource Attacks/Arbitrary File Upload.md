2025-08-21 19:27

Status:

Tags: [[eWPT]] [[File & Resource Attacks]]
###### Prerequisites: 
# Arbitrary File Upload
## Overview

- Vulnerability in web apps allowing attackers to upload and execute malicious files.
    
- Consequences: unauthorized access, server compromise, full system control.
    
- Cause: improper validation of uploaded files (e.g., type checks, location restrictions).
    

## Exploitation

- Attacker uploads malicious file (e.g., PHP script, shell).
    
- Uses bypass tricks like misleading extensions (`.php.jpg`).
    

## Impact

- **Remote Code Execution (RCE):** attacker gains control of server.
    
- **Data Exfiltration:** theft/manipulation of sensitive data.