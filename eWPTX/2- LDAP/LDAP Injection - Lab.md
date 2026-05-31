2026-02-04 06:29

Status:

Tags:[[eWPTX]][[LDAP]]
###### Prasyarat:
# LDAP Injection - Lab
Di lab ini kita latihan LDAP Injection dari INE (vuLnDAP): https://github.com/digininja/vuLnDAP

**Step 1:** Buka link lab untuk mengakses instance Kali (GUI).

![[Pics/INE/1_a6e2d311eb.png]]

**Step 2:** Cek apakah mesin/domain target bisa dijangkau.

**Command:**

```
ping -c3 demo.ine.local
```

![[Pics/INE/2_241ccacae9.png]]

Mesin target dapat dijangkau.

**Step 3:** Cek port yang terbuka di mesin target.

**Command:**

```
nmap -p- demo.ine.local
```

![[Pics/INE/3_9654e95641.png]]

Port 22 (SSH) dan 9090 terbuka. Sesuai deskripsi challenge, web app yang rentan ada di port 9090.

**Step 4:** Akses web application di port 9090.

Buka URL berikut di browser:

**URL:** http://demo.ine.local:9090

![[Pics/INE/4_ec31e9cce0.png]]

Terlihat instance [vuLnDAP](https://github.com/digininja/vuLnDAP) pada mesin target.

**Step 5:** Eksplor web application.

Klik menu **Stock Control**:

![[Pics/INE/5_b7fb641609.png]]

Pilih kategori **Fruit**:

![[Pics/INE/5_1_48e1fdc208.png]]

Perhatikan URL:

![[Pics/INE/5_2_c2c5fa99f7.png]]

Nilai **fruits** direfleksikan pada parameter **objectClass**.

# Eksploitasi LDAP injection pada web application

**Step 6:** Lakukan LDAP injection.

Set nilai `*` pada parameter **objectClass**:

**Catatan:** `*` adalah karakter spesial (wildcard) yang bisa membuat query mengembalikan semua item yang cocok.

![[Pics/INE/6_792a957cbd.png]]

![[Pics/INE/6_1_7f99fc73f2.png]]

Output juga memuat nama user sistem.

Klik **More Info** untuk administrator (user `david`):

![[Pics/INE/6_2_581ef67495.png]]

Klik **Back**:

![[Pics/INE/6_3_199859aa8e.png]]

Perhatikan parameter **objectClass** sekarang berisi nilai **posixAccount**.

Halaman ini berisi daftar user sistem yang tersimpan di LDAP database.

**Step 7:** Ambil password SSH untuk user `david` dari LDAP database.

Click on **More Info** link for the administrator (user david):

![[Pics/INE/7_237fdc7bbe.png]]

Halaman hasilnya belum menampilkan informasi yang menarik:

![[Pics/INE/7_1_f3accc2944.png]]

Cek referensi schema LDAP di URL berikut:

**URL:** https://tldp.org/HOWTO/archived/LDAP-Implementation-HOWTO/schemas.html

![[Pics/INE/7_2_7b070b5984.png]]

Perhatikan object class **posixaccount**. Beberapa atribut yang umum ada:

- uidNumber
- gidNumber
- homedirectory
- userpassword

Buka home page web app yang rentan:

**URL:** http://demo.ine.local:9090

![[Pics/INE/7_3_2194d31f43.png]]

Admin menyimpan SSH keys di database.

Cari info terkait SSH keys pada object class posixaccount:

**Search Query:**  

```
ldap posixaccount ssh keys
```

![[Pics/INE/7_4_e968c72d29.png]]

Buka link [StackOverflow](https://serverfault.com/questions/653792/ssh-key-authentication-using-ldap) dari hasil pencarian:

![[Pics/INE/7_5_6f1abe5ae8.png]]

Kita bisa memakai atribut `sshPublicKey` untuk mengambil SSH keys.

Pakai URL berikut untuk mengambil `uidNumber`, `gidNumber`, `homedirectory`, `userpassword`, dan `sshPublicKey` untuk user `david`:

**URL:** http://demo.ine.local:9090/item?cn=david&disp=uidNumber,gidNumber,homedirectory,userpassword,sshPublicKey

![[Pics/INE/7_6_3ac1be1d0e.png]]

Perhatikan `sshPublicKey` justru mengembalikan password (bukan SSH key). Kemungkinan ini adalah password SSH user tersebut.

**Kemungkinan password SSH untuk user david:** r0ck_s0l1d_p4ssw0rd

**Step 8:** Ambil flag dari mesin target memakai kredensial SSH yang didapat.

Coba konek ke target via SSH dengan kredensial berikut:

**Username:** david  
**Password:** r0ck_s0l1d_p4ssw0rd

**Command:**

```
ssh david@demo.ine.local
```

![[Pics/INE/8_adc3da8e0d.png]]

Login berhasil.

Ambil flag:

**Commands:**  

```
ls
cat flag
```

![[Pics/INE/8_1_5ce7691a25.png]]

**Flag:** 5520dd2d85e5003db92048c629bb5072

Selesai untuk lab LDAP injection ini.