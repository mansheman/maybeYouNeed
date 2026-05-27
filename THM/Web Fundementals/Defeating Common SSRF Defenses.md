2025-07-16 17:55

Status:

Tags: [[THM Web Fundementals]] [[tryhackme]]
###### Prerequisites: 
# Defeating Common SSRF Defenses
## Deny List

- **Concept** – accept all outbound requests _except_ those matching blocked hosts, IPs, or patterns.
    
- **Common entries to block**
    
    - `localhost`, `127.0.0.1`
        
    - Alternate localhost forms: `0`, `0.0.0.0`, `0000`, `127.1`, `127.*.*.*`, `2130706433`, `017700000001`
        
    - Cloud‑metadata endpoint: `169.254.169.254`
        
- **Typical bypasses**
    
    - Sub‑domains that resolve to blocked IPs, e.g. `127.0.0.1.nip.io`
        
    - DNS records pointing to `169.254.169.254` under attacker‑controlled domain
        

## Allow List

- **Concept** – deny every request unless the destination matches an explicit safe host/pattern.
    
- **Example rule**: URL must start with `https://website.thm/`
    
- **Typical bypass**: chain a sub‑domain after the allowed host, e.g. `https://website.thm.attacker-domain.thm/`, which still satisfies a naive prefix check.
    

## Open Redirect Trick

When both deny‑ and allow‑list checks fail to bypass:

1. Find an **open redirect** on the target site, e.g.  
    `https://website.thm/link?url=https://tryhackme.com`
    
2. Supply the redirect endpoint as the request URL. It passes the allow‑list (because it begins with `https://website.thm/`).
    
3. The server fetches the redirect URL internally and is then forwarded to an attacker‑controlled domain.
    

> [!info] **Key Takeaway** Combine strict allow‑list validation, blocklists for sensitive internal addresses, and eliminate open redirects to harden against SSRF.