2026-02-06

Status:

Tags: [[eWPTX]] [[Bypassing Filters & Evasion]]
###### Prerequisites: [[Bypassing Filters & Evasion]]
# WAF Fingerprinting & Detection

## Why Fingerprint a WAF

Before attempting bypass techniques, identify what you're up against. Different WAFs have different detection logic, default rules, and known weaknesses.

---

## Automated Detection

### wafw00f

The standard tool for WAF identification:

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

## Manual Fingerprinting

### Response header analysis

Look for these headers in responses:

| Header | WAF |
|--------|-----|
| `Server: cloudflare` | Cloudflare |
| `X-Sucuri-ID` | Sucuri |
| `X-CDN: Imperva` | Imperva/Incapsula |
| `Server: AkamaiGHost` | Akamai |
| `X-Powered-By: AWS Lambda` | AWS WAF |
| `Server: BIG-IP` | F5 BIG-IP |

### Cookie-based identification

| Cookie name | WAF |
|-------------|-----|
| `__cfduid`, `cf_clearance` | Cloudflare |
| `visid_incap_*`, `incap_ses_*` | Imperva |
| `sucuri_cloudproxy_*` | Sucuri |
| `ak_bmsc` | Akamai |

### Behavioral fingerprinting

Send an obvious attack payload and observe:

```bash
curl -v "https://target.com/?id=1' OR 1=1--"
```

Observe the response:

- **Cloudflare**: 403 with "Attention Required!" or "Ray ID" in body
- **ModSecurity**: 403 with "ModSecurity" or "OWASP CRS" in error message
- **AWS WAF**: 403 with `{"message":"Forbidden"}`
- **Akamai**: "Access Denied" with reference number
- **Imperva**: "Request unsuccessful" with incident ID

### Error page analysis

Trigger different error codes and compare responses:

```bash
# Try a long URL
curl "https://target.com/$(python3 -c 'print("A"*10000)')"

# Try special characters
curl "https://target.com/<script>alert(1)</script>"

# Try a path traversal
curl "https://target.com/../../etc/passwd"
```

---

## Common WAFs and Their Characteristics

| WAF | Detection Logic | Known Weaknesses |
|-----|----------------|------------------|
| **ModSecurity + CRS** | Regex-based rule matching, anomaly scoring | Rule-specific bypasses, encoding tricks |
| **Cloudflare** | ML + signature-based hybrid | Unicode normalization, chunked encoding |
| **AWS WAF** | Configurable rules (managed + custom) | Depends on ruleset configuration |
| **Akamai** | Behavioral analysis + signatures | Request fragmentation |
| **Imperva** | ML + signatures + behavioral | Multipart/form-data tricks |
| **F5 BIG-IP ASM** | Positive/negative security model | Parameter pollution |

---

## Next Steps After Identification

1. Research known bypasses for the specific WAF version
2. Test [[Encoding & Transformation Bypasses]]
3. Apply WAF-specific [[WAF Bypass Payloads & Techniques]]
4. Check [[Filter Bypass Case Studies]] for real-world examples
