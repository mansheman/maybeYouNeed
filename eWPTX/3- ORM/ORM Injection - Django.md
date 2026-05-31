2026-02-06

Status:

Tags: [[eWPTX]] [[ORM]]
###### Prasyarat: [[ORM Injection]]
# Injeksi ORM — Django

## Gambaran singkat

Django ORM umumnya aman karena query ter-parameterisasi. Namun, ada method tertentu yang menerima raw SQL atau memungkinkan manipulasi filter sehingga bisa memicu injection.

---

## Method yang berisiko

### raw()

Menjalankan raw SQL — vektor injection paling jelas:

```python
# VULNERABLE
User.objects.raw(f"SELECT * FROM auth_user WHERE username = '{username}'")

# SAFE — parameterized
User.objects.raw("SELECT * FROM auth_user WHERE username = %s", [username])
```

### extra()

Menambahkan potongan raw SQL ke query (deprecated di Django 4.0+):

```python
# VULNERABLE
User.objects.extra(where=[f"username = '{user_input}'"])

# SAFE
User.objects.extra(where=["username = %s"], params=[user_input])
```

### annotate() with RawSQL

```python
from django.db.models.expressions import RawSQL

# VULNERABLE
queryset.annotate(val=RawSQL(f"SELECT col FROM table WHERE id = {user_input}", []))

# SAFE
queryset.annotate(val=RawSQL("SELECT col FROM table WHERE id = %s", [user_input]))
```

---

## QuerySet Filter Injection

Django ORM memakai lookup double-underscore (`__`). Kalau nama field dikontrol user, query bisa dimanipulasi:

```python
# If field_name comes from user input
field_name = request.GET.get('sort')  # e.g., "password"
User.objects.filter(**{field_name: value})

# Attacker sends: sort=password
# Result: User.objects.filter(password=value)
# This leaks whether a password matches — boolean oracle
```

### Lookup injection

```python
# Attacker sends: field=username__startswith
# Result: User.objects.filter(username__startswith="a")
# Can enumerate values character by character
```

**Dampak**: ekstraksi data berbasis boolean lewat manipulasi filter.

---

## Mass assignment

```python
# VULNERABLE — accepts all POST data as model fields
user = User(**request.POST.dict())
user.save()
# Attacker sends: is_staff=true&is_superuser=true

# SAFE — explicit field selection
user = User(
    username=request.POST['username'],
    email=request.POST['email']
)
```

---

## Mitigasi

- Jangan pakai `raw()`, `extra()`, atau `RawSQL` dengan input yang tidak disanitasi
- Selalu gunakan query ter-parameterisasi (`%s` + params list)
- Jangan pernah memakai input user sebagai nama field di `filter(**kwargs)`
- Gunakan Django forms/serializers untuk validasi + whitelist field
- Pakai `only()` atau `defer()` untuk membatasi field yang di-query
