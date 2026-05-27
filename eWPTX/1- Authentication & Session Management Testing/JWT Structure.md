2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[JWT]]
###### Prerequisites: [[JSON Web Tokens (JWT)]]
# JWT Structure

## Header

Metadata about algorithm and type:

```json
{"alg":"HS256","typ":"JWT"}
```

## Payload

Claims about the subject/session:

```json
{"sub":"123","name":"John Doe","admin":true,"exp":1716454560}
```

## Signature

Prevents tampering (created by signing header+payload with a key).

Key point:

- if header/payload change, signature must fail validation

---

## Diagram

![[Pics/auth/jwt-structure.svg]]
