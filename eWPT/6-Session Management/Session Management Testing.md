2025-08-18 14:14

Status:

Tags: [[eWPT]] [[Session Management]]
###### Prerequisites: 
# Session Management Testing

## Overview
- Critical part of web application security testing.  
- Evaluates how effectively and securely a web app manages user sessions.  
- Helps identify vulnerabilities that may lead to breaches, unauthorized access, or data leakage.  

## Key Tests

### 1. Session Fixation
- Attempt to set a known session ID before login.  
- Login with another account and check if the application accepts the predefined session ID.  
- Vulnerable apps may let attackers hijack valid sessions.  

### 2. Session Hijacking
- Capture and reuse another user’s session ID.  
- Tools: **Wireshark**, **Burp Suite**.  
- Test if intercepted IDs allow access to the victim’s account.  

### 3. Session ID Brute-Force
- Try to guess or brute-force valid session IDs.  
- Assess ID randomness and resistance to attacks.  
