2026-02-04 23:32

Status:

Tags: [[eWPTX]] [[Web Services]]
###### Prasyarat: [[Web Services]] [[SOAP]] [[WSDL (Web Services Description Language)]]
# Lab Web Service - 1 - SOAP User Account (WSDL + Metode Tersembunyi)

## Sumber

- [[Web Services Lab]]

---

**Langkah 1:** Buka link lab untuk mengakses instance GUI Kali.

![[Pics/INE/1_1ce97ec19d.png]]

**Langkah 2:** Cek apakah mesin/domain yang diberikan bisa dijangkau.

**Perintah:**

```
ping -c3 demo.ine.local
```

![[Pics/INE/2_ca5d6b61f9.png]]

Mesin yang diberikan bisa dijangkau.

**Langkah 3:** Cek port yang terbuka pada target.

**Perintah:**

```
nmap -sS -sV demo.ine.local
```

![[Pics/INE/3_6c71a77e24.png]]

Port 80 (Apache web server) dan 3306 (MySQL server) terbuka di target.

**Langkah 4:** Buka browser untuk menginspeksi website yang di-host.

**URL:** http://demo.ine.local

![[Pics/INE/4_8bd2e93727.png]]

Ada instance [**OWASP Mutillidae II**](https://github.com/webpwnized/mutillidae) yang berjalan di Apache.

**Langkah 5:** Buka web service **Lookup User**.

Di deskripsi challenge, link web service sudah disediakan.

Kamu bisa menuju web service lewat menu di web:

Masuk ke **Web Services** > **SOAP** > **Username Enumeration** > **Lookup User**:

![[Pics/INE/5_bfedbc98d6.png]]

Alternatifnya, langsung buka URL web service dari deskripsi challenge:

**URL:** http://demo.ine.local/webservices/soap/ws-user-account.php

Dan kamu akan masuk ke halaman berikut:

![[Pics/INE/5_1_232b83e120.png]]

**Langkah 6:** Enumerasi file WSDL.

**Info:**
WSDL adalah singkatan dari Web Services Description Language, digunakan untuk mendeskripsikan web service. WSDL ditulis dalam XML.

Dokumen WSDL mendeskripsikan web service: lokasi service dan method/operation, melalui elemen-elemen utama berikut:

![[Pics/INE/6_3ac6dc4620.png]]

**Referensi:** https://www.w3schools.com/xml/xml_wsdl.asp

Makanya file ini menarik: ini “kontrak” web service yang akan kita pentest.

Kita akan cari semua operation/method yang didefinisikan beserta parameter yang diterima, untuk nanti kita panggil manual.

Di halaman web service, klik link **WSDL** atau tambahkan `?wsdl` di URL:

![[Pics/INE/6_1_074df822bd.png]]

Itu akan menampilkan file WSDL untuk web service yang akan kita interaksikan:

![[Pics/INE/6_2_1c5b7370db.png]]

Seperti yang dijelaskan di [W3schools WSDL page](https://www.w3schools.com/xml/xml_wsdl.asp), `<portType>` mendeskripsikan operation yang bisa dilakukan web service beserta message/parameter yang harus dikirim.

Kalau kamu cek WSDL-nya, service ini mendukung 5 operation:

- getUser
- createUser
- updateUser
- getAdminInfo
- deleteUser

![[Pics/INE/6_3_c78a3e0e14.png]]

![[Pics/INE/6_4_f56f972c18.png]]

Dokumentasi untuk semua operation tersebut juga tersedia di file WSDL.

Kesimpulannya: halaman web service hanya menampilkan 3 dari 5 operation. Kemungkinan 2 sisanya tidak ditujukan untuk user biasa dan “disimpan” untuk admin.

**Langkah 7:** Cek informasi operation `getUser`.

Kembali ke halaman web service lalu klik link `getUser`:

![[Pics/INE/7_791e278cac.png]]

Terlihat detail operation `getUser`, termasuk parameter input dan output. Informasi yang sama juga bisa ditemukan di WSDL.

Scroll ke bawah untuk melihat request lengkap untuk memanggil method `getUser`:

![[Pics/INE/7_1_399db4b8f6.png]]

**Langkah 8:** Jalankan Burp Suite.

Buka start menu lalu pilih: **03 - Web Application Analysis** -> **burpsuite**

![[Pics/INE/8_259809fec7.png]]

Kalau ada warning soal versi JDK, abaikan saja lalu klik **OK**:

![[Pics/INE/8_1_ecde6f6ec2.png]]

Buat project sementara:

![[Pics/INE/8_2_5493a91a93.png]]

Gunakan konfigurasi default Burp Suite:

![[Pics/INE/8_3_9b607210bd.png]]

Setelah itu Burp akan terbuka.

![[Pics/INE/8_4_ec2d5f7ef4.png]]

**Langkah 8:** Panggil method `getUser`.

Buka tab Repeater di Burp Suite:

![[Pics/INE/9_fdb559d4df.png]]

Copy request pemanggilan `getUser` dari halaman web, lalu paste ke Repeater:

![[Pics/INE/9_1_40633912a0.png]]

Hapus bagian `/mutillidae` dari URL karena web service berada di `/webservice` (bukan `/mutillidae/webservice`).

Setelah itu, kirim request.

Biasanya akan muncul dialog untuk mengisi `Host` dan `Port` target:

**Host:** demo.ine.local **Port:** 80

![[Pics/INE/9_2_5ff00fbb8c.png]]

Setelah target di-set, klik send lagi:

![[Pics/INE/9_3_473cd7cc10.png]]

Request berhasil.

Scroll response dan perhatikan `username` serta `signature` yang dikembalikan:

![[Pics/INE/9_4_2c4ae63ff1.png]]

**Langkah 10:** Panggil method `deleteUser`.

Ganti semua kemunculan `getUser` menjadi `deleteUser`, lalu kirim request:

![[Pics/INE/10_cf622d02c8.png]]

Kita mendapat response 200.

Kalau scroll, terlihat error:

![[Pics/INE/10_1_6b66ca90e8.png]]

Error tersebut menunjukkan ada parameter yang kurang pada request.

Namun yang lebih penting: kita berhasil memanggil operation yang tadinya tidak ditampilkan.

Kembali ke WSDL dan cari `deleteUserRequest`:

![[Pics/INE/10_2_2e0e2266fc.png]]

Di situ terlihat function ini menerima dua parameter: `username` dan `password` (keduanya bertipe string).

**Langkah 11:** Melakukan serangan SQL Injection.

Sekarang kita tahu `deleteUser` butuh dua parameter, tapi kita tidak tahu password user mana pun.

Bisa melakukan dictionary attack, atau cek apakah web service rentan SQL Injection.

Kita coba opsi kedua. Pertama, kirim single quote (`'`) sebagai password untuk melihat apakah muncul error:

![[Pics/INE/11_16792e5412.png]]

Kita tetap dapat response 200.

Kalau scroll, muncul SQL error:

![[Pics/INE/11_1_abceff4303.png]]

Error message-nya bahkan membocorkan query SQL yang dieksekusi:

**SQL Query:**

```
SELECT username FROM accounts WHERE username='Jeremy' AND password=''';
```

Seperti terlihat, ada satu quote tambahan (`'`) yang membuat query SQL-nya jadi tidak valid.

Jadi cukup jelas ada celah SQL Injection di web service ini.

Sekarang kirim payload SQLi berikut:

```
' or '1'='1
```

Payload di atas akan menghasilkan query berikut:

**SQL Query:**

```
SELECT username FROM accounts WHERE username='Jeremy' AND password='' or '1'='1';
```

Kondisi di sisi kanan `OR` akan selalu true karena `'1'='1'`. Jadi walau kondisi kiri false, hasil akhirnya tetap true.

Karena itu, payload di field password seharusnya menghapus akun user bernama `Jeremy`:

![[Pics/INE/11_2_3e0d4988c8.png]]

Scroll untuk melihat pesan penghapusan akun:

![[Pics/INE/11_3_f0e40af825.png]]

Kita juga mendapatkan flag pertama:

**flag1:** 6701f2d8bf691da5ee694d3a1786a7e6

**Langkah 12:** Panggil method `getAdminInfo`.

Cek WSDL untuk melihat parameter yang dibutuhkan oleh `getAdminInfo`:

![[Pics/INE/12_34c910169d.png]]

Terlihat bahwa `getAdminInfo` tidak butuh parameter.

Scroll untuk melihat dokumentasi `getAdminInfo`:

![[Pics/INE/12_1_680fa4e0b4.png]]

Method ini mengambil beberapa detail akun (non-sensitive) milik admin. Contoh request juga disediakan.

Masuk ke Burp Repeater dan kirim request berikut:

**Request:**

```
POST /webservices/soap/ws-user-account.php HTTP/1.1
Accept-Encoding: gzip,deflate
Content-Type: text/xml;charset=UTF-8
Content-Length: 387
Host: localhost
Connection: Keep-Alive
User-Agent: Apache-HttpClient/4.1.1 (java 1.5)
<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:ws-user-account">
<soapenv:Header/>
<soapenv:Body>
<urn:getAdminInfo soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
</urn:getAdminInfo>
</soapenv:Body>
</soapenv:Envelope>
```

![[Pics/INE/12_2_8ab6955d27.png]]

Request tersebut menghasilkan response 500.

Kalau dilihat lebih lanjut, tampaknya hanya admin yang bisa memanggil method ini:

![[Pics/INE/12_3_a7a9bad9ff.png]]

Jadi memang ada pembatasan pada pemanggilan method ini.

**Langkah 13:** Bypass pembatasan SOAP body dengan header SOAPAction.

Kemungkinan ada restriksi saat memanggil `getAdminInfo`.

Salah satu cara lain memanggil method SOAP adalah dengan header SOAPAction.

**Info:**
SOAPAction adalah header transport (HTTP atau JMS) yang dikirim bersama pesan SOAP untuk memberi informasi “intent” request ke service. Interface WSDL mendefinisikan nilai SOAPAction untuk tiap operation. Beberapa implementasi menggunakan SOAPAction untuk menentukan behavior.

**Referensi:** https://www.ibm.com/docs/en/baw/19.x?topic=binding-protocol-headers

Kembali ke WSDL lalu cek atribut `soapaction` untuk operation `getAdminInfo`:

![[Pics/INE/13_36ef5150b4.png]]

**SOAPAction attribute value:**  

```
urn:ws-user-account#getAdminInfo
```

Sekarang di Burp Repeater, kirim request berikut:

**Request:**

```
POST /webservices/soap/ws-user-account.php HTTP/1.1
Accept-Encoding: gzip,deflate
Content-Type: text/xml;charset=UTF-8
Content-Length: 228
Host: localhost
Connection: Keep-Alive
User-Agent: Apache-HttpClient/4.1.1 (java 1.5)
SOAPAction: urn:ws-user-account#getAdminInfo
<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:ws-user-account">
</soapenv:Envelope>
```

Perhatikan: kita mengirim header SOAPAction dan menghapus SOAP header/body pada request XML.

![[Pics/INE/13_1_fdef6b42cf.png]]

Request tersebut menghasilkan response 200.

Scroll response untuk melihat detail user admin dan flag kedua:

![[Pics/INE/13_2_f0dd8e6229.png]]

**flag2:** 4315749f21a66fad53895daccbebf309

**Sampai sini: kita sudah enumerasi WSDL, memanggil method tersembunyi, melakukan SQL injection, dan bypass pembatasan SOAP body.**

