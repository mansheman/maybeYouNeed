2025-05-23 14:05

Status:

Tags: [[eWPT]]  [[Web Fingerprinting and Enumeration]] #Information_Gathering
###### Prerequisites:

[[Netcraft]]

# DNS enumration
## dnsdumpster.com is the best website for this manner
# Tool Deep Dive: dnsrecon

**Course:** Web Enumeration & Information Gathering
**Main Topic Focus:** Using dnsrecon for DNS Enumeration


**Suggested File Path for this note:** `eWPT/Web Fingerprinting and Enumeration/`

---

## dnsrecon - DNS Enumeration Tool

**dnsrecon** is a powerful Python script used for DNS enumeration. It helps discover various DNS records associated with a domain, identify potential misconfigurations like open zone transfers, and find subdomains through techniques like brute-forcing.

### Basic Usage and Common Options

The general syntax for `dnsrecon` is:
`dnsrecon -d <target_domain> [options]`

**Commonly Used Options:**

*   `-d, --domain <domain>`: **Required.** Specifies the target domain to enumerate.
*   `-t, --type <type(s)>`: Specifies the type of enumeration to perform or record types to query.
    *   `std`: Performs a standard enumeration (A, AAAA, SOA, NS, MX, TXT). This is often a good starting point.
    *   `axfr`: Attempts a DNS Zone Transfer.
    *   `brt`: Brute-forces subdomains using a wordlist.
    *   `srv`: Queries for SRV records.
    *   `all`: Attempts all enumeration types (can be very noisy and time-consuming).
    *   You can also specify individual record types: `A`, `MX`, `NS`, etc.
*   `-D, --dictionary <file>`: Path to a custom wordlist file for brute-forcing subdomains (used with `-t brt`).
*   `-n <ip_address>`: Specify an alternative DNS server to use for queries.
*   `--xml <filename.xml>`: Save output to an XML file.
*   `-v`: Verbose output.

### Example Command

Let's say we want to perform a standard enumeration and attempt a zone transfer against `example.com`:

```bash
dnsrecon -d example.com -t std,axfr
````

content_copydownload

Use code [with caution](https://support.google.com/legal/answer/13505487).Markdown

If we also wanted to brute-force subdomains using a common wordlist (assuming one is at /usr/share/wordlists/dnsmap.txt):

```
dnsrecon -d example.com -t std,axfr,brt -D /usr/share/wordlists/dnsmap.txt
```

content_copydownload

Use code [with caution](https://support.google.com/legal/answer/13505487).Bash

### Example Output & Explanation

(This is a simplified and illustrative example of what dnsrecon might output)

```
[*] Performing General Enumeration for: example.com
[*] DNS Servers:
    ns1.example-dns.com
    ns2.example-dns.com
[*] A Records:
    example.com       93.184.216.34
    www.example.com   93.184.216.34
[*] AAAA Records:
    example.com       2606:2800:220:1:248:1893:25c8:1946
[*] MX Records:
    mail.example.com (10)
    mail2.example.com (20)
[*] NS Records:
    ns1.example-dns.com
    ns2.example-dns.com
[*] SOA Record:
    Primary NS: ns1.example-dns.com
    Admin Email: hostmaster.example.com
    Serial: 2023102701
    Refresh: 7200
    Retry: 3600
    Expire: 1209600
    Minimum TTL: 3600
[*] TXT Records:
    example.com       "v=spf1 include:_spf.example.com ~all"
    _dmarc.example.com "v=DMARC1; p=reject; rua=mailto:dmarc-rua@example.com"

[*] Attempting Zone Transfer (AXFR) for example.com on ns1.example-dns.com...
[!] Zone Transfer Failed: Transfer refused or not supported.
[*] Attempting Zone Transfer (AXFR) for example.com on ns2.example-dns.com...
[!] Zone Transfer Failed: Transfer refused or not supported.

[*] Brute Forcing Subdomains with dictionary /usr/share/wordlists/dnsmap.txt:
    [+] Found: dev.example.com (93.184.216.100)
    [+] Found: test.example.com (93.184.216.101)
    [+] Found: internal.example.com (10.0.5.25)
    ...
```

content_copydownload

Use code [with caution](https://support.google.com/legal/answer/13505487).

**Explanation of Output Sections:**

- **DNS Servers:** Lists the nameservers identified for the domain.
    
- **A/AAAA Records:** IP addresses (IPv4/IPv6) associated with the domain and any discovered subdomains through standard lookups. www is often an A record pointing to the same IP as the apex domain.
    
- **MX Records:** Mail exchange servers responsible for handling email for the domain, with their priority (lower number means higher priority).
    
- **NS Records:** The authoritative name servers for the domain. These are the primary servers to query for domain information.
    
- **SOA Record:** Start of Authority record provides administrative details about the zone: primary nameserver, admin contact, serial number (can indicate how often the zone changes), and various timers.
    
- **TXT Records:** Text records can contain various information, such as SPF (Sender Policy Framework) for email validation, DMARC policies, site verification codes, etc.
    
- **Zone Transfer (AXFR):**
    
    - **Failed:** This is common and good security practice by the target. It means the DNS server did not allow a full dump of its zone records.
        
    - **Successful (Hypothetical):** If successful, it would list ALL DNS records for the domain, potentially revealing many internal and external hostnames and their IPs. This is a significant information disclosure.
        
- **Brute Forcing Subdomains:** Lists subdomains discovered by trying common names from the provided dictionary. The IP addresses are resolved for each found subdomain.
    

### Key Takeaways for Enumeration (What's Important to Note)

- **IP Address Space:** The A and AAAA records provide direct IP addresses for the main domain and any immediately resolvable common subdomains (like www). These are initial points for Network Scanning and Service Discovery.
    
- **Nameservers (NS):** Knowing the authoritative nameservers is crucial. These are the servers you'd target for a DNS Zone Transfer Vulnerability attempt. Different NS might have different configurations.
    
- **Mail Servers (MX):** Identifying mail servers can be useful for:
    
    - Understanding the email infrastructure.
        
    - Potential targets for Phishing Attacks (if in scope).
        
    - Servers that might be configured differently or have different patch levels.
        
- **Subdomains (from BRT, AXFR):** This is often the goldmine.
    
    - Each new subdomain represents an expansion of the **attack surface**.
        
    - Look for interesting names like dev, test, staging, uat, vpn, internal, admin, api, etc., as these might be less secured or expose different applications/services.
        
    - Pay special attention to subdomains resolving to **internal IP addresses** (e.g., 10.x.x.x, 172.16.x.x-172.31.x.x, 192.168.x.x) if discovered via AXFR or brute-forcing. This could indicate DNS Rebinding possibilities or split-horizon DNS.
        
- **TXT Records for Security Insights:**
    
    - **SPF/DKIM/DMARC** records give insights into email security configurations. Weak SPF records might allow email spoofing.
        
    - Other TXT records might accidentally leak version numbers, internal hostnames, or other metadata.
        
- **Zone Transfer (AXFR) Results:**
    
    - **Success:** A critical finding. It provides a comprehensive map of the target's DNS, potentially revealing dozens or hundreds of hosts, including internal ones, dramatically expanding the scope of hosts to investigate.
        
    - **Failure:** Indicates proper security hardening against zone transfers, which is a good sign for the target's DNS security.
        
- **SOA Record Information:**
    
    - The admin email (hostmaster.example.com) could be a target for social engineering.
        
    - The primary nameserver is confirmed here.
        

