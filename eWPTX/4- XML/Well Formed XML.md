2026-02-04 17:34

Status:

Tags: [[eWPTX]] [[XML]]
###### Prasyarat: [[XML Syntax]]
# XML yang Well Formed

Dokumen XML dengan sintaks yang benar disebut **well formed**.

Aturan utama:

- hanya satu root element
- tag bersifat case sensitive (`<Letter>` ≠ `<letter>`)
- closing tag wajib ada (tidak boleh “lupa” menutup)
- XML prolog sifatnya opsional, tapi kalau ada harus berada paling awal

---

## Well formed vs valid

- **Well formed**: aturan sintaks benar
- **Valid**: well formed DAN tervalidasi terhadap DTD/schema
