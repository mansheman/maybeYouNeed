2025-10-14 13:36

Status:

Tags:[[eWPTX]] [[Authentication]]
###### Prerequisites: 
## Authentication Mechanisms

Authentication mechanisms are methods or processes used to verify the identity of a user or system attempting to access a web application or service.

These mechanisms ensure that only authorized users can gain access to sensitive resources, enhancing security.

In the following sections, we explore some of the key types of authentication mechanisms used in web applications.

---

## Types of Authentication Mechanisms

### Password-Based Authentication

Users provide a username and a password to verify their identity.

---

### Multi-Factor Authentication (MFA)

Combines two or more independent credentials, such as:

- Something you know (password or PIN).
    
- Something you have (smartphone, security token).
    
- Something you are (biometric verification like fingerprint or facial recognition).
    

---

### Two-Factor Authentication (2FA)

A specific type of MFA that requires exactly two factors for authentication, often a password and a one-time code sent via SMS or an authenticator app.

---

### Token-Based Authentication

Uses tokens (e.g., JSON Web Tokens or OAuth tokens) that are issued upon successful login and used for subsequent requests, reducing the need to repeatedly enter credentials.

---

### Single Sign-On (SSO)

Allows users to log in once and gain access to multiple applications or services without needing to re-enter credentials, often using protocols like SAML or OAuth.

---

### One-Time Passwords (OTP)

A temporary password sent to the user via SMS or email for a single login session, often used in conjunction with other authentication methods.