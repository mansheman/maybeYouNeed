2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prasyarat: [[XML Injection - Overview]]
# XML Tag Injection - Gambaran

## Definition

**XML Tag Injection** adalah jenis XML injection ketika penyerang menyisipkan atau memodifikasi **tag XML**, sehingga struktur dokumen berubah.

---

## Key characteristics

- fokusnya pada tag (bukan hanya value teks)
- dapat mengganggu parsing atau memicu logika tak terduga
- bisa berujung privilege escalation jika server memercayai tag yang disisipkan

---

## Example (concept)

Kalau aplikasi menerima XML yang bisa dikontrol user, lalu memutuskan permission berdasarkan isi tag, tag yang disisipkan dapat mengubah keputusan tersebut.
