2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prasyarat: [[XML Injection - Overview]]
# XML Injection - Jenis Serangan

## Jenis issue injection / abuse parser yang umum di XML

|Tipe serangan|Apa yang dimanipulasi|Dampak umum|
|---|---|---|
|XML data manipulation|nilai/struktur XML|data tampering, auth bypass|
|XML Tag Injection|tag baru/diubah|privilege escalation, parsing issues|
|XML Attribute Injection|atribut|privilege escalation, data tampering|
|XPath Injection|ekspresi XPath untuk query XML|akses data tanpa izin, auth bypass|
|XXE|external entities / fitur DTD|file disclosure, SSRF, DoS|
|Billion Laughs|entity rekursif|DoS (resource exhaustion)|
|XML bombs|XML besar/dalam|DoS|
|XInclude Injection|include eksternal|file inclusion, data disclosure|
|XML Schema poisoning|manipulasi schema/validasi|bypass validasi, data tak terduga|
