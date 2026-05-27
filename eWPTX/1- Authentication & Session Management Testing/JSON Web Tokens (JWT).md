2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[JWT]]
###### Prerequisites: [[Token-Based Authentication]]
# JSON Web Tokens (JWT)

## Overview

JWTs are compact, URL-safe tokens used for authentication/authorization and information exchange.

Structure:

```text
HEADER.PAYLOAD.SIGNATURE
```

- header: algorithm + token type
- payload: claims (user/session data)
- signature: integrity/authenticity

Important:

- payload is Base64-encoded, not encrypted
- don’t put secrets (passwords) in JWT claims
