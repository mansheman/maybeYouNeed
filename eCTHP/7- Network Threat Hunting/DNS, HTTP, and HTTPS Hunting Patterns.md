2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Network IOCs and Triggers]]
# DNS, HTTP, and HTTPS Hunting Patterns

## DNS

Suspicious signals:

- queries to non-DNS servers
- repeated queries with no responses
- responses without matching queries
- very long or encoded-looking subdomains

## HTTP

Suspicious signals:

- HTTP on unusual ports
- encrypted-looking traffic on port 80
- direct external IP connections instead of normal FQDN use
- repetitive beacon-like behavior

## HTTPS

Suspicious signals:

- plaintext traffic on port 443
- unexpected protocol behavior after a supposed TLS connection
- highly regular external encrypted sessions

## Diagram

![[Pics/ecthp/dns-http-https-abnormality.svg|1200]]
