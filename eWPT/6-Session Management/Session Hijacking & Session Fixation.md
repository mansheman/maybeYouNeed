2025-08-18 14:57

Status:

Tags: [[eWPT]] [[Session Management]]
###### Prerequisites: 
# Session Hijacking & Session Fixation

## Session Hijacking
- Also called **session theft**.  
- Attacker takes over a user’s active session by stealing their **session token/ID**.  
- Allows impersonation of the victim and unauthorized access.  

### Token Acquisition
- **Session Prediction**: Guessing weak or predictable session tokens.  
- **Session Sniffing**: Intercepting tokens over unsecured networks (e.g., open Wi-Fi).  
- **XSS Attack**: Injecting malicious JavaScript to steal tokens from the victim’s browser.  

### Impersonation
- Attacker presents the stolen token to the application.  
- Application treats attacker as the authenticated user.  

### Impact
- **Data Theft**: Access to sensitive info (personal, financial, confidential).  
- **Account Takeover**: Change passwords, settings, or lock out the victim.  
- **Malicious Transactions**: Unauthorized purchases or actions.  
- **Data Manipulation**: Modify or delete user data.  

---

## Session Fixation
- Attack where the attacker **fixes a session token** to a known value.  
- Victim is tricked into logging in with that token, giving the attacker access.  

### Token Acquisition
- **Predictable Tokens**: Exploit weak/randomness in token generation.  
- **Intercepting Tokens**: Capturing tokens sent insecurely (e.g., no HTTPS).  

### Impersonation
- Attacker forces victim to use the attacker’s chosen session token.  
- Methods:  
  - Malicious URL with fixed token.  
  - Manipulated link.  
  - Social engineering.  

### Hijacking
- Once the victim logs in with the attacker’s token, the attacker gains full access.  
- Application recognizes the attacker as the legitimate user.  
