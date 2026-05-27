2025-05-24 23:26

Status:

Tags: [[eWPT]]  [[Web Fingerprinting and Enumeration]] #Information_Gathering
###### Prerequisites:



# OWAS amass
**OWASP Amass for Automated Web Server Fingerprinting Recon**

Amass automates the discovery of targets (domains, subdomains, IPs) for subsequent web server fingerprinting. It doesn't fingerprint servers itself but feeds tools like Nmap and Dirb.

**Key Amass Integration Points:**

1.  **`amass intel`**: Gathers initial OSINT (domains, ASNs) for an organization.
    *   Usage: `amass intel -org "OrgName"`

2.  **`amass enum`**: Core enumeration engine to find subdomains and IPs.
    *   **`-passive`**: Uses only passive techniques (no direct target traffic).
    *   **`-src`**: Shows the data source for each finding.
    *   **`-ip`**: Resolves discovered names to IP addresses.
    *   **`-dir /path/to/output`**: Specifies an output directory for Amass results (graph database).
    *   Combined Passive Example: `amass enum -passive -src -ip -d example.com -dir ./amass_results`

3.  **`amass viz`**: Visualizes collected data from the `-dir` output, helping identify asset relationships.
    *   Usage: `amass viz -d3 -dir ./amass_results`

**Automated Workflow Outline:**

1.  **Discover (Amass):** Use `intel` then `enum` (with `-passive`, `-ip`, `-dir`) to get IPs/hostnames.
2.  **Fingerprint (Nmap, etc.):** Script Nmap (`-sV`) against Amass-found IPs to identify web server versions.
3.  **Enumerate Dirs (Dirb, etc.):** Script directory brute-forcers against identified web servers.

Amass focuses on finding *what* to fingerprint, especially using passive methods initially, with `-dir` storing its findings for subsequent tools and `viz` for analysis.