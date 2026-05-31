2026-02-04 06:27

Status:

Tags: [[eWPTX]] [[LDAP]] [[LDAP Injection]]
###### Prasyarat: [[LDAP Injection - Overview]] [[LDAP Injection - Examples]]
# LDAP Injection - Testing Methodology

## Gambaran

Testing LDAP injection biasanya **berbasis perilaku (behavior-based)**:

- jumlah hasil berbeda
    
- konten halaman berbeda (mis. “no results” vs “results”)
    
- redirect / hasil autentikasi berbeda
    
- kadang muncul error LDAP eksplisit (jarang di produksi)
    
---

## 1) Cari endpoint yang kemungkinan LDAP-backed

Target yang probabilitasnya tinggi:

- login
    
- directory search / autocomplete
    
- access control berbasis group
    
- lookup “pilih departemen/printer/device”
    

Clue di response:

- atribut seperti `uid`, `cn`, `sn`, `mail`, `memberOf`
    
- referensi ke AD / OpenLDAP / “directory”
    

---

## 2) Identifikasi parameter yang mempengaruhi filter

Input umum yang sering berakhir di filter LDAP:

- username / email
    
- pencarian nama
    
- selector departemen / OU
    
- tipe device/printer
    

---

## 3) Cari sinyal “filter break”

Kirim variasi input yang “normal” lalu amati:

- hasil lebih luas dari yang seharusnya
    
- hasil tiba-tiba kosong
    
- perilaku true/false yang stabil (sinyal blind)
    
- error server-side (syntax / invalid filter)
    

Kalau punya log (pengujian internal), cari juga:

- string filter LDAP persis yang dieksekusi
    

---

## 4) Konfirmasi dampak (prioritaskan)

Konfirmasi dengan urutan berikut:

1. **Authorization bypass** (check group/role)
2. **Authentication bypass** (perilaku login)
3. **Data exposure** (entry/atribut berlebih)
4. **Enumeration** (cek eksistensi, timing)

---

## 5) Dokumentasikan seperti catatan SQLi

Yang perlu dicatat:

- endpoint + parameter yang vuln
    
- expected vs observed behavior
    
- pola filter LDAP yang dipakai aplikasi (kalau terlihat / bisa diinfer)
    
- screenshot / response
    
- dampak security (auth, authz, data)
    
- rekomendasi perbaikan (escaping + allow-list + least privilege)
