2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prasyarat: [[XML Injection - Overview]]
# XML Injection - Metodologi Testing

## Tujuan

Konfirmasi apakah input user bisa **memecahkan** (break) atau **membentuk ulang** (reshape) dokumen XML dan menghasilkan perubahan perilaku yang bermakna.

---

## 1) Identify XML entry points

- endpoint yang menerima `application/xml` atau SOAP
- fitur import/upload XML
- parameter yang disisipkan ke template XML server-side

---

## 2) Probe with XML metacharacters

Metacharacter yang sering relevan untuk testing XML injection:

- `'` dan `"` (sering mempengaruhi value atribut)
- `<` dan `>` (batas tag)
- `&` (entity)

Kalau aplikasi membangun XML secara tidak aman, karakter ini bisa memicu:

- parsing exceptions
- different response paths
- different authorization outcomes

---

## 3) Quotes (single/double)

Quote membentuk value atribut.

Kalau input user ditempatkan di dalam atribut tanpa encoding, quote injection bisa memutus atribut dan mengubah dokumen.

---

## 4) Ampersand (`&`)

`&` begins an entity reference like:

```text
&EntityName;
```

Ide testing:

- sisipkan nama entity palsu untuk melihat apakah parser error
- coba format entity yang rusak (tanpa `;`) untuk melihat cara parsing gagal

---

## 5) Angle brackets (`<` `>`)

Angle brackets define tags/comments/CDATA:

```xml
<tagname>value</tagname>
<!-- comment -->
<![CDATA[value]]>
```

Ide testing:

- sisipkan `<` atau `>` dan cek parse error
- cari perubahan perilaku yang mengindikasikan input masuk ke konteks XML

---

## Yang perlu didokumentasikan (seperti catatan SQLi)

- di mana XML diterima/dibangun
- expected vs observed behavior
- pesan error (jika ada)
- kategori dampak (auth, authz, data exposure, DoS)
- rekomendasi fix (contextual encoding + allow-list + parser aman)
