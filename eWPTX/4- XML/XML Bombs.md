2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prasyarat: [[XML Injection - Attack Types]]
# XML Bombs

## Gambaran singkat

XML bombs adalah payload denial-of-service yang membebani parser lewat XML yang sangat besar atau nested terlalu dalam. Berbeda dengan [[Billion Laughs Attack]], XML bombs bersifat **non-recursive** — tidak mengandalkan rantai entity expansion. Mereka lebih mengandalkan ukuran/repetisi (brute force).

## Quadratic blowup attack

Mendefinisikan satu entity besar lalu mereferensikannya ribuan kali. Tidak ada rekursi, tapi output tetap membengkak.

```xml
<?xml version="1.0"?>
<!DOCTYPE bomb [
  <!ENTITY a "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa">  <!-- 40 chars -->
]>
<bomb>
  &a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;
  &a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;
  <!-- repeat entity reference ~50,000 times -->
</bomb>
```

**Math**: entity 40-byte diulang 50.000 kali = ~2 MB teks hasil ekspansi dari input ~500 KB. Pertumbuhannya kuadratik (O(n^2)), bukan eksponensial.

Ini bisa bypass proteksi Billion Laughs yang hanya mengecek ekspansi rekursif/nested, tetapi tidak menghitung jumlah repetisi.

## Attribute blowup

Membebani parser dengan ribuan atribut pada satu elemen:

```xml
<element
  a1="val" a2="val" a3="val" a4="val" a5="val"
  a6="val" a7="val" a8="val" a9="val" a10="val"
  <!-- ... thousands more attributes ... -->
  a9999="val" a10000="val"
/>
```

Ini memaksa parser mengalokasikan entry (mis. di hash table) untuk setiap atribut. Beberapa parser melakukan lookup O(n^2) untuk cek atribut duplikat, sehingga CPU bisa terkuras.

## Deep nesting bomb

Deeply nested elements without entity expansion:

```xml
<a><a><a><a><a><a><a><a><a><a><a><a><a><a><a><a><a><a><a><a>
<!-- 10,000+ levels deep -->
</a></a></a></a></a></a></a></a></a></a></a></a></a></a></a></a></a></a></a></a>
```

Memicu stack overflow pada parser yang parsing-nya rekursif. Efektif pada parser yang tidak punya limit kedalaman.

## Ringkasan konsumsi resource

| Tipe serangan     | Ukuran input | Dampak memori | Dampak CPU | Bypass limit rekursi |
|-------------------|-----------|---------------|------------|---------------------------|
| Quadratic blowup  | ~500 KB   | ~2 MB+        | Moderate   | Yes                       |
| Attribute blowup  | ~200 KB   | Moderate      | High       | Yes                       |
| Deep nesting      | ~100 KB   | Stack-based   | High       | Yes                       |
| Billion Laughs    | ~1 KB     | ~3 GB         | High       | No                        |

## Billion Laughs vs XML bombs

- **Billion Laughs**: eksponensial, entity rekursif, input kecil, ekspansi masif
- **XML Bombs**: linear/kuadratik, tanpa rekursi, input lebih besar, tetap efektif untuk DoS
- XML bombs berguna saat parser punya limit entity expansion tapi tidak punya limit ukuran/kedalaman

## Hardening parser

1. **Batasi ukuran dokumen** — tolak XML di atas ambang (mis. 1 MB)
2. **Batasi kedalaman elemen** — limit nesting (mis. 100–200 level)
3. **Batasi jumlah atribut per elemen** — cap (mis. 100–200)
4. **Batasi entity expansion** — mencakup bombs dan Billion Laughs
5. **Pasang parsing timeout** — hentikan parsing setelah waktu wajar
6. **Pakai streaming parser (SAX/StAX)** — proses elemen per elemen (tidak memuat DOM penuh)
