2026-02-04 22:04

Status:

Tags: [[eWPTX]] [[Web Services]] [[XML]]
###### Prasyarat: [[SOAP]]
# WSDL (Web Services Description Language)

## Gambaran singkat

WSDL adalah bahasa berbasis XML untuk mendeskripsikan fungsi dan interface sebuah web service.

Ia berperan sebagai **kontrak** antara provider dan consumer, yang menjelaskan:

- operasi
- struktur input/output
- binding (cara pesan dikirim)
- endpoint (alamat service)

WSDL paling sering ditemui di service SOAP.

---

## Versi WSDL

- WSDL 1.1
- WSDL 2.0 (lebih baru, tapi 1.1 masih sangat umum)

---

## Definisi abstrak vs konkret

- **Abstract**: apa yang dilakukan service (operasi + message)
- **Concrete**: bagaimana/di mana service disediakan (binding + endpoint)

---

## Komponen (gambaran umum)

- **Types**: mendefinisikan tipe data (biasanya schema XSD)
- **Message**: struktur pesan (WSDL 1.1)
- **PortType**: daftar operasi yang didukung (WSDL 1.1)
- **Binding**: detail protokol/encoding
- **Service**: info endpoint (URL)

Catatan WSDL 2.0:

- memakai **Interface** sebagai pengganti `portType`
- tidak memakai `message` dengan cara yang sama; lebih langsung merujuk ke elemen schema
