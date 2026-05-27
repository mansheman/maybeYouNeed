2025-08-24 08:30

Status:

Tags: [[eWPT]] [[File & Resource Attacks]]
###### Prerequisites: 

# Directory Traversal

## Overview

- Directory traversal (also known as **path traversal** or **directory climbing**) is a security vulnerability that occurs when a web application allows access to files and directories outside the intended or authorized directory scope.
    
- If unmitigated, it can lead to **data breaches** and **system compromise**.
    

---

## Causes (Exploitation)

- **Improper Input Handling**: Application fails to validate or sanitize user input when dealing with file paths (e.g., URL parameters).
    
- **Attacker Manipulation**: Input is altered with special sequences to escape intended directories.
    

---

## Technique

- **Traversing Directories**: Attackers use `../` (dot-dot) sequences to move up the directory structure.
    
- **Accessing Sensitive Files**: After escaping, attackers can read or manipulate sensitive files (e.g., config files, user data, system files).
    

---

## Impact

- **Unauthorized Data Access**: Exposure of sensitive information (credentials, configs, source code).
    
- **Data Leakage**: Leakage of confidential or private data (financial records, IP, user data).
    
- **System Compromise**: In severe cases, attackers may gain arbitrary code execution, privilege escalation, or full system control.
    

---

## Example

- Vulnerable file download functionality:
    
    ```
    http://example.com/download?file=user123.txt
    ```
    
- Attacker attempts traversal:
    
    ```
    http://example.com/download?file=../../../../../etc/passwd
    ```
    
- If unchecked, the server returns the `/etc/passwd` file (Unix system accounts).
    

---

## Methodology

1. Attacker appends `../` multiple times to move up the directory tree.
    
2. Reaches the root directory (`/`).
    
3. Requests sensitive files (e.g., `/etc/passwd`).
    
4. If no validation/sanitization exists, the file is exposed.
    

---

## Directory Traversal Basics

- Insecure input handling leads to unintended file access.
    
- Always validate and sanitize file path inputs.
    
- Restrict file access to **whitelisted directories only**.
    

---
