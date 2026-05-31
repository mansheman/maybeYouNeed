---
date: 2025-12-29 19:12
status:
tags:
tags2: "[[Authentication]]"
Tag3: "[[eWPTX]]"
---

# Bypass CAPTCHA

## Gambaran Singkat
Tes ini fokus untuk mengidentifikasi **mekanisme lockout yang lemah**, khususnya implementasi **CAPTCHA** yang tujuannya mencegah login otomatis (bot) tapi desainnya buruk atau enforcement-nya tidak konsisten.

---

## Apa itu CAPTCHA?
**CAPTCHA** (Completely Automated Public Turing test to tell Computers and Humans Apart) adalah kontrol keamanan yang meminta user menyelesaikan challenge untuk membuktikan bahwa dia manusia, sehingga mengurangi serangan bot.

---

## Prerequisites
- Paham alur autentikasi
- Familiar dengan brute force / automation
- Dasar scripting
- Paham konsep OCR dan serangan berbasis ML

---

## Jenis CAPTCHA

---

## 1. Arithmetic-Based CAPTCHA (Weak)

**Deskripsi:**
- Meminta user menjawab soal aritmatika sederhana (mis. *“3 + 7 = ?”*).

**Kekuatan:**
- Sangat lemah

**Teknik bypass:**
- Otomasi/brute force
- Layanan pemecah CAPTCHA (mis. 2Captcha)

**Kenapa gagal:**
- Kompleksitas sangat rendah
- Mudah dipecahkan script/bot
- Tidak benar-benar membedakan manusia vs bot


---

## 2. Text-Based CAPTCHA (Basic)

**Deskripsi:**
- Menampilkan teks yang didistorsi/diacak dan user harus mengetiknya dengan benar.

**Kekuatan:**
- Basic

**Observasi:**
- Sedikit lebih kompleks dibanding aritmatika
- Sering ada noise/background atau huruf terdistorsi

**Kerentanan:**
- Bot dengan OCR / model ML bisa membaca teks
- Distorsi sederhana mudah ditaklukkan tool modern
- Sering dibypass massal


---

## 3. Image-Based CAPTCHA (Moderate)

**Deskripsi:**
- User memilih gambar yang sesuai kondisi  
  (e.g., *“Select all images with traffic lights”*).

**Kekuatan:**
- Moderate

**Kenapa lebih baik:**
- Butuh model AI yang lebih canggih
- Lebih sulit diautomasi dibanding text-based

**Kerentanan:**
- Tetap bisa dilawan dengan machine learning
- Model object recognition bisa bypass jika training cukup
- Tidak 100% bot-proof


---

## 4. reCAPTCHA v2 (Moderate → Strong)

**Deskripsi:**
- Dikembangkan oleh Google
- Umumnya berupa klik **“I’m not a robot”**
- Bisa memunculkan challenge gambar tergantung risk

**Catatan keamanan:**
- Menggabungkan interaksi user + analisis perilaku
- Umumnya lebih sulit dibypass dibanding CAPTCHA buatan sendiri

---

## 5. reCAPTCHA v3 (Very Strong)

**Deskripsi:**
- CAPTCHA “invisible”
- Berjalan di background tanpa interaksi user
- Memberi **risk score** berdasarkan perilaku

**Kenapa kuat:**
- Tidak ada puzzle yang bisa “dipecahkan”
- Sulit diautomasi dengan meyakinkan
- Bertumpu pada sinyal perilaku, bukan teka-teki

---

## Key Takeaway
> CAPTCHA alone is **not sufficient** as a lockout mechanism.  
> Implementasi yang lemah bisa dibypass dengan mudah, terutama jika tidak ada:
- Rate limiting
- Kebijakan lockout akun
- Analisis perilaku
- Validasi server-side yang benar

---
# Lab Documentation
file:///Users/abdalrahmanmajed/Downloads/walkthrough-2172.pdf
