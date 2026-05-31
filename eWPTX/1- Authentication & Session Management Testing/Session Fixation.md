2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[Session Management]]
###### Prasyarat: [[Session Management Testing]]
# Session Fixation

## Definisi

Session fixation terjadi saat penyerang memaksa korban memakai session ID yang sudah diketahui penyerang, lalu session ID itu “naik level” setelah korban login—dan kemudian dipakai ulang oleh penyerang.

---

## Cara kerja (high level)

1) attacker sets/acquires a session ID
2) victim logs in while using that same session ID
3) app associates that session ID with the authenticated session
4) attacker reuses the known session ID to access the victim session

---

## Penyebab umum

- not regenerating session ID on login
- allowing session IDs in URLs or client-controlled locations
- weak/predictable session ID generation
- missing cookie flags (`Secure`, `HttpOnly`, `SameSite`)
- long-lived sessions without expiry/timeout

---

## Ide pengujian

- log in and check whether session ID changes after authentication
- test whether the app accepts session IDs in URL/query
- verify logout invalidates the session server-side
