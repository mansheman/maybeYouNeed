2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]]
###### Prerequisites: [[Authentication]]
# Token-Based Authentication

## Overview

Token-based authentication is a method to validate and authorize users/applications using tokens that clients present with requests.

Unlike classic server-stored sessions, token approaches are often more **stateless** (especially for APIs).

---

## Types of tokens

### Bearer tokens

- whoever holds the token can use it
- common for APIs
- must be protected and short-lived

### JSON Web Tokens (JWT)

- self-contained token: header + payload (claims) + signature
- widely used for stateless authentication

### OAuth tokens

- access tokens / refresh tokens
- used in OAuth2 flows for authorization
