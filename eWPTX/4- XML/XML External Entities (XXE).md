2026-02-04 20:43

Status:

Tags: [[eWPTX]] [[XML]]
###### Prasyarat: [[XML Injection]] [[XML DTD (Document Type Definition)]]
# XML External Entities (XXE)

## Gambaran singkat

Jenis XML injection yang paling berbahaya sering melibatkan penyisipan **external entity** ke definisi dokumen XML. Ini dikenal sebagai **XXE (XML eXternal Entities)**.

XXE terjadi ketika XML parser memproses external entity yang direferensikan oleh dokumen XML secara tidak aman. Dampaknya bisa berupa:

- **Data exposure** (reading sensitive server files)
- **SSRF** (accessing internal/external resources from the server)
- **Denial of Service (DoS)** (resource exhaustion via entities/features)

---

## Ide inti

Intinya, penyerang berusaha “menyuruh” parser memuat entity dari luar, sehingga bisa membaca konten sensitif di host atau membuat server melakukan request jaringan.

---

## External entity: private vs public (konsep)

Secara konsep, ada dua jenis external entity:

- **Private external entities**: biasanya terbatas untuk pemakaian tertentu (author/group). Ini sering paling berbahaya karena bisa mereferensikan file lokal atau resource internal.
- **Public external entities**: untuk pemakaian lebih luas/sharing, biasanya menunjuk identifier/resource publik.

Practical takeaway: file disclosure dan SSRF paling sering datang dari pemrosesan private entity.

![[Pics/Screenshot 2026-02-04 at 8.44.28 PM.png]]

---

## Teknik: resource inclusion (file disclosure)

Skenario: penyerang bisa meng-upload/menyusun dokumen XML (atau aplikasi membangun XML yang mengandung DTD yang dikontrol penyerang), dan parser mengizinkan resolusi external entity.

### 1) Define an external entity (file)

```xml
<!ENTITY xxefile SYSTEM "file:///etc/passwd">
```

### 2) Reference the entity in the XML body

```xml
<!DOCTYPE message [
  <!ENTITY xxefile SYSTEM "file:///etc/passwd">
]>
<message>
  <body>&xxefile;</body>
</message>
```

### 3) Observe the output channel

Untuk mengonfirmasi disclosure, kamu butuh perilaku aplikasi yang memantulkan nilai hasil parse (mis. error message, field yang dirender, log yang bisa kamu baca, response API, dll).

![[Pics/Screenshot 2026-02-04 at 8.44.59 PM.png]]
---

## Catatan (MOC)

- [[XXE - Lab (Apache Solr)]]
- [[Blind & OOB XXE]]

## Diagram

![[Pics/xml/xxe-attack-types.svg]]
