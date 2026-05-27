2026-02-06

Status:

Tags: [[eWPTX]] [[ORM]]
###### Prerequisites: [[ORM Injection]]
# ORM Injection — Django

## Overview

Django ORM is generally safe due to parameterized queries. However, certain methods accept raw SQL or allow filter manipulation that can lead to injection.

---

## Dangerous Methods

### raw()

Executes raw SQL — most obvious injection vector:

```python
# VULNERABLE
User.objects.raw(f"SELECT * FROM auth_user WHERE username = '{username}'")

# SAFE — parameterized
User.objects.raw("SELECT * FROM auth_user WHERE username = %s", [username])
```

### extra()

Adds raw SQL fragments to the query (deprecated in Django 4.0+):

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

Django's ORM uses double-underscore (`__`) lookups. If the field name is user-controlled, an attacker can manipulate the query:

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

**Impact**: Boolean-based data extraction via filter manipulation.

---

## Mass Assignment

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

## Mitigation

- Never use `raw()`, `extra()`, or `RawSQL` with unsanitized input
- Always use parameterized queries (`%s` with params list)
- Never use user input as field names in `filter(**kwargs)`
- Use Django forms or serializers to validate and whitelist fields
- Use `only()` or `defer()` to control which fields are queried
