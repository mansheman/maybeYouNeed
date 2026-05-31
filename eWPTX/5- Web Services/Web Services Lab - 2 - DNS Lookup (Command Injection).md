2026-02-04 23:32

Status:

Tags: [[eWPTX]] [[Web Services]]
###### Prasyarat: [[Web Services]] [[SOAP]]
# Lab Web Service - 2 - DNS Lookup (Injeksi Perintah)

## Sumber

- [[Web Services Lab]]

---
## Melakukan serangan command injection pada web service DNS Lookup

**Langkah 14:** Buka web service DNS Lookup.

Masuk ke **Web Services** > **SOAP** > **Command Injection** > **DNS Lookup**:

![[Pics/INE/14_b5afa313a1.png]]

Kamu akan masuk ke halaman berikut:

![[Pics/INE/14_1_3206d32edd.png]]

**Langkah 15:** Cek file WSDL.

Klik link WSDL untuk melihat file WSDL dari web service ini:

![[Pics/INE/15_36b1e50737.png]]

Terlihat bahwa `lookupDNS` adalah satu-satunya operation yang tersedia:

![[Pics/INE/15_1_863a52ea01.png]]

Lihat detail operation `lookupDNS`:

![[Pics/INE/15_2_c04a7dfac1.png]]

Halaman web menampilkan parameter input dan output yang diharapkan. Informasi yang sama juga bisa ditarik langsung dari WSDL.

Scroll untuk melihat contoh request:

![[Pics/INE/15_3_8fbcbad25a.png]]

**Langkah 16:** Interaksi dengan web service memakai Burp Suite.

Copy sample request untuk method `lookupDNS` ke Burp Repeater:

![[Pics/INE/16_0c51cce4ac.png]]

Hapus bagian `/mutillidae` dari URL karena web service berada di `/webservice` (bukan `/mutillidae/webservice`).

Setelah itu, kirim request:

![[Pics/INE/16_1_0a92371c82.png]]

Kita mendapat response 200.

Scroll untuk melihat hasil DNS Lookup:

![[Pics/INE/16_2_a4ecc73e8f.png]]

**Langkah 17:** Identifikasi celah command injection.

Kirim tanda titik-koma (`;`) sebagai ganti hostname:

![[Pics/INE/17_92e46bbd0d.png]]

Scroll untuk melihat hasil DNS Lookup:

![[Pics/INE/17_1_d307a9ee03.png]]

Perhatikan bahwa input dipakai “apa adanya” dan tidak muncul error.

Kirim payload berikut ke web service:

**Payload:**

```
;ls -al
```

Payload di atas menghasilkan response 200:

![[Pics/INE/17_2_28d8e6840c.png]]

Scroll untuk melihat hasil DNS Lookup:

![[Pics/INE/17_3_3ae6977164.png]]

Perhatikan output dari perintah `ls -al` ikut muncul.

Artinya web service ini memang rentan command injection.

**Langkah 18:** Ambil flag.

Karena web service DNS Lookup rentan command injection, kirim perintah berikut untuk mencari file flag:

**Payload:**

```
;find / -iname *flag* 2>/dev/null
```

Payload di atas menghasilkan response 200:

![[Pics/INE/18_4db1f01410.png]]

Output menunjukkan flag ketiga berada di file `/app/flag3`.

Kirim payload berikut untuk membaca file flag:

**Payload:**

```
;cat /app/flag3
```

![[Pics/INE/18_1_a8869ab80d.png]]

**flag3:** 6ae5422a0f086335766e1de8f75c16b7

**Sampai sini: kita sudah melakukan serangan command injection pada web service.**

Dengan ini lab selesai. Intinya: kita belajar cara enumerasi service SOAP berbasis WSDL, menguji input yang dieksekusi server-side, dan memverifikasi impact lewat respons.
