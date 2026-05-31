2026-02-05 06:17

Status:

Tags: [[eWPTX]]
###### Prasyarat: [[Bypassing Filters & Evasion]]
# Regex (Regular Expression)

## Gambaran singkat

**Regex** adalah “bahasa pola” untuk mencocokkan (dan kadang mengganti) teks.

Di security, regex sering muncul di:

- WAF rules
- input validation filters
- log parsing/detection
- allow-lists / deny-lists

---

## Kenapa penting untuk bypass filter

Banyak “filter” pada praktiknya hanyalah regex.

Kalau kamu paham regex-nya, kamu bisa memprediksi:

- apa yang diblok
- apa yang diizinkan
- di mana celahnya (anchor, greedy, case sensitivity, Unicode, dll.)

---

## Dasar cepat

### Common tokens

- `.` karakter apa pun (kecuali newline di banyak engine)
- `*` nol atau lebih
- `+` satu atau lebih
- `?` opsional (nol atau satu)
- `{m,n}` rentang pengulangan
- `^` awal string/baris
- `$` akhir string/baris
- `|` OR (atau)
- `()` group / capture
- `(?:...)` non-capturing group
- `[]` character class
  - `[abc]` one of a/b/c
  - `[a-z]` range
  - `[^a-z]` NOT range
- `\d` digit, `\w` word char, `\s` whitespace (detail bisa beda antar engine)

### Contoh

Cocokkan pola email (sederhana):

```regex
^[^@\s]+@[^@\s]+\.[^@\s]+$
```

Cocokkan `admin` atau `root` (flag case-insensitive biasanya di-set di luar pattern):

```regex
^(admin|root)$
```

Cocokkan teks yang mengandung `select` (contoh filter keyword SQLi yang naif):

```regex
select
```

---

## Cara cepat ngetes filter (disarankan)

Copy regex-nya ke regex101 untuk melihat penjelasan pattern dan hasil match.

- Site: https://regex101.com

Tips saat pakai regex101:

- pilih “flavor” regex yang benar (PCRE / Python / JavaScript)
- uji input yang lolos dan yang diblok
- cari anchor yang hilang (`^`/`$`) dan pattern yang terlalu longgar

---

## Kesalahan regex yang sering bikin bypass

- tanpa anchor: pattern match substring, bukan seluruh input
- catastrophic backtracking: risiko regex DoS
- memakai deny-list alih-alih allow-list
- normalisasi tidak konsisten (URL decode, Unicode, case folding) antara filter dan backend
