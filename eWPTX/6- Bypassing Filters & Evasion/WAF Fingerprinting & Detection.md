2026-02-06

Status:

Tags: [[eWPTX]] [[Bypassing Filters & Evasion]]
###### Prasyarat: [[Bypassing Filters & Evasion]]
# Fingerprinting & Deteksi WAF

## Kenapa perlu fingerprint WAF?

Sebelum mencoba teknik bypass, identifikasi dulu WAF apa yang kamu hadapi. Tiap WAF punya logika deteksi, default rule, dan kelemahan yang berbeda.

---

## Deteksi otomatis

### wafw00f

Tool standar untuk identifikasi WAF:

```bash
# Basic scan
wafw00f https://target.com

# Verbose output
wafw00f -v https://target.com

# Test all WAFs (don't stop at first match)
wafw00f -a https://target.com

# List all detectable WAFs
wafw00f -l
```

### Nmap WAF scripts

```bash
# HTTP WAF detection
nmap -p 443 --script http-waf-detect target.com

# HTTP WAF fingerprint
nmap -p 443 --script http-waf-fingerprint target.com
```

---

## Fingerprinting manual

### Response header analysis

Cari header berikut di response:

| Header | WAF |
|--------|-----|
| `Server: cloudflare` | Cloudflare |
| `X-Sucuri-ID` | Sucuri |
| `X-CDN: Imperva` | Imperva/Incapsula |
| `Server: AkamaiGHost` | Akamai |
| `X-Powered-By: AWS Lambda` | AWS WAF |
| `Server: BIG-IP` | F5 BIG-IP |

### Cookie-based identification

| Nama cookie | WAF |
|-------------|-----|
| `__cfduid`, `cf_clearance` | Cloudflare |
| `visid_incap_*`, `incap_ses_*` | Imperva |
| `sucuri_cloudproxy_*` | Sucuri |
| `ak_bmsc` | Akamai |

### Behavioral fingerprinting

Kirim payload yang jelas “mencurigakan” lalu amati responnya:

```bash
curl -v "https://target.com/?id=1' OR 1=1--"
```

Amati responnya:

- **Cloudflare**: 403 dengan "Attention Required!" atau "Ray ID" di body
- **ModSecurity**: 403 dengan "ModSecurity" atau "OWASP CRS" di error message
- **AWS WAF**: 403 dengan `{"message":"Forbidden"}`
- **Akamai**: "Access Denied" dengan reference number
- **Imperva**: "Request unsuccessful" dengan incident ID

### Error page analysis

Picu berbagai tipe error dan bandingkan responnya:

```bash
# Try a long URL
curl "https://target.com/$(python3 -c 'print("A"*10000)')"

# Try special characters
curl "https://target.com/<script>alert(1)</script>"

# Try a path traversal
curl "https://target.com/../../etc/passwd"
```

---

## WAF umum dan karakteristiknya

| WAF | Logika deteksi | Kelemahan umum |
|-----|----------------|------------------|
| **ModSecurity + CRS** | Rule matching berbasis regex, anomaly scoring | Bypass spesifik rule, trik encoding |
| **Cloudflare** | Hybrid ML + signature | Unicode normalization, chunked encoding |
| **AWS WAF** | Rule dapat dikonfigurasi (managed + custom) | Tergantung konfigurasi ruleset |
| **Akamai** | Behavioral analysis + signature | Fragmentasi request |
| **Imperva** | ML + signature + behavioral | Trik multipart/form-data |
| **F5 BIG-IP ASM** | Model keamanan positif/negatif | Parameter pollution |

---

## Langkah lanjut setelah teridentifikasi

1. Riset bypass yang dikenal untuk versi WAF terkait
2. Uji [[Encoding & Transformation Bypasses]]
3. Terapkan teknik spesifik WAF di [[WAF Bypass Payloads & Techniques]]
4. Cek [[Filter Bypass Case Studies]] untuk contoh dunia nyata

Praktik yang baik: dokumentasikan hasilnya (payload mana yang diblok, HTTP status, body/error string), karena itu akan jadi baseline saat iterasi bypass.
