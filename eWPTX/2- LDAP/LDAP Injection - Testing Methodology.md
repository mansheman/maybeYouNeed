2026-02-04 06:27

Status:

Tags: [[eWPTX]] [[LDAP]] [[LDAP Injection]]
###### Prerequisites: [[LDAP Injection - Overview]] [[LDAP Injection - Examples]]
# LDAP Injection - Testing Methodology

## Overview

LDAP injection testing is usually **behavior-based**:

- different results count
    
- different page content (e.g., “no results” vs “results”)
    
- different redirects / auth outcome
    
- sometimes explicit LDAP errors (rare in production)
    
---

## 1) Find likely LDAP-backed endpoints

High probability targets:

- login
    
- directory search / autocomplete
    
- group-based access controls
    
- “choose department/printer/device” lookup
    

Clues in responses:

- attributes like `uid`, `cn`, `sn`, `mail`, `memberOf`
    
- references to AD / OpenLDAP / “directory”
    

---

## 2) Identify parameters that influence a filter

Common inputs that end up in LDAP filters:

- username / email
    
- name search
    
- department / OU selector
    
- device/printer type
    

---

## 3) Look for “filter break” signals

Submit benign variations and watch for:

- broader results than expected
    
- empty results unexpectedly
    
- stable true/false behavior (blind signal)
    
- server-side errors (syntax / invalid filter)
    

If you have logs (internal test), also look for:

- the exact LDAP filter string being executed
    

---

## 4) Confirm impact (prioritize)

Confirm in this order:

1. **Authorization bypass** (group/role checks)
2. **Authentication bypass** (login behavior)
3. **Data exposure** (extra entries/attributes)
4. **Enumeration** (existence checks, timing)

---

## 5) Document like your SQLi notes

Capture:

- vulnerable endpoint + parameter
    
- expected vs observed behavior
    
- the LDAP filter pattern used by the app (if visible / inferred)
    
- screenshots / responses
    
- security impact (auth, authz, data)
    
- fix recommendation (escaping + allow-list + least privilege)
