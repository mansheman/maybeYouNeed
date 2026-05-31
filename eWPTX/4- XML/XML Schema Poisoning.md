2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prasyarat: [[XML DTD (Document Type Definition)]]
# XML Schema Poisoning

## Gambaran singkat

Schema poisoning mengacu pada manipulasi schema/asumsi validasi supaya XML yang berbahaya atau tidak terduga tetap lolos validasi.

Impact:

- bypass validasi
- penyisipan data tak terduga yang mempengaruhi logika aplikasi

Mitigasi (level tinggi):

- gunakan schema yang strict dan dipin (versi/path jelas)
- jangan fetch schema dari lokasi yang tidak tepercaya
- tegakkan validasi + limit keamanan parser
