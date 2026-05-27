2025-05-24 15:14

Status:

Tags: [[eWPT]]  [[Web Fingerprinting and Enumeration]] #Information_Gathering
###### Prerequisites:



# Web Server fingerprinting
it is the process of finding the technology that the web server is using
we do it to now what stack it uses
also to know the exact version of the server for example apache  because we can search for vulnerabilities for that version


Okay, here's an Obsidian note based on your terminal screenshots, focusing on web server fingerprinting and enumeration.


# Web Server Fingerprinting & Enumeration: demo.ine.local (Apache 2.4.18)

**Date:** 2025-05-24
**Target IP:** `192.16.169.3`
**Target Hostname (from Nmap):** `demo.ine.local`

## 1. Initial Port Scanning & Service Version Detection (Nmap)

The first step was to identify open ports and the services running on them.

**Command:**
```bash
nmap -sV -p- 192.16.169.3

*(Note: The screenshot also showed `nmap -sV -p 80 --script=http-enum 192.16.169.3` which is more targeted after initial discovery, and `nmap -sV -f 192.16.169.3` using fragmented packets.)*

**Key Findings:**
*   **Port 80/tcp:** Open
*   **Service:** HTTP
*   **Version:** Apache httpd 2.4.18 ((Ubuntu))
*   **MAC Address:** `02:42:C0:10:A9:03` (less relevant for web fingerprinting itself but good general recon info)

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
       |_http-server-header: Apache/2.4.18 (Ubuntu)
```

## 2. Web Server Enumeration (Nmap HTTP Scripts)

Using Nmap's scripting engine to find common web directories and files.

**Command:**
```bash
nmap -sV -p 80 --script=http-enum 192.16.169.3
```

**Key Findings from `http-enum`:**
*   `robots.txt`: Robots file found.
    *   `http-enum: /robots.txt: Robots file`
*   Potentially interesting directories with listing enabled (according to Nmap's interpretation):
    *   `/data/`: `Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'`
    *   `/dir/`: `Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'`
    *   `/src/`: `Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'`

## 3. Directory Brute-Forcing (Dirb)

Confirming and discovering further directories using a wordlist.

**Command:**
```bash
dirb http://192.16.169.3 /usr/share/metasploit-framework/data/wordlists/directory.txt
```

**Key Findings from `dirb`:**
*   `http://192.16.169.3/data` (CODE:301 SIZE:311) - Confirms `/data/` found by Nmap. The 301 redirect suggests it likely redirects to `/data/`.
*   `http://192.16.169.3/dir` (CODE:301 SIZE:310) - Confirms `/dir/` found by Nmap. The 301 redirect suggests it likely redirects to `/dir/`.

```
---- Scanning URL: http://192.16.169.3/ ----
+ http://192.16.169.3/data (CODE:301 SIZE:311)
+ http://192.16.169.3/dir (CODE:301 SIZE:310)
```

## 4. Vulnerability Research (SearchSploit)

Based on the identified Apache version, checked for known public exploits.

**Command:**
```bash
searchsploit apache 2.4.18
```

**Key Potential Vulnerability:**
*   **Apache 2.4.17 < 2.4.38 - 'apache2ctl graceful' 'Logrotate' Local Privilege Escalation**
    *   Path: `linux/local/46676.php`
    *   **Note:** This is a *Local* Privilege Escalation (LPE), meaning it requires initial access to the system to be exploited. It's not a remote code execution vulnerability against the web server itself from an unauthenticated perspective.

## Summary of Fingerprinted Information

*   **Web Server Software:** Apache
*   **Version:** 2.4.18
*   **Operating System (Hinted):** Ubuntu
*   **Key Discovered Paths:**
    *   `/robots.txt`
    *   `/data/` (confirmed, 301 redirect)
    *   `/dir/` (confirmed, 301 redirect)
    *   `/src/` (potential)
*   **Potential Vulnerabilities:** At least one known LPE for Apache 2.4.18 exists.

## Next Steps (Hypothetical)

1.  **Manually inspect:**
    *   `http://192.16.169.3/robots.txt` for disallowed entries that might reveal more paths.
    *   `http://192.16.169.3/data/`
    *   `http://192.16.169.3/dir/`
    *   `http://192.16.169.3/src/`
    *   Look for directory listings or any exposed files.
2.  **Further directory/file brute-forcing:** Use tools like `gobuster` or `ffuf` with more comprehensive wordlists and extension fuzzing.
3.  **Technology-specific checks:** Research common vulnerabilities or misconfigurations for Apache 2.4.18 on Ubuntu.
4.  **If initial access is gained:** The LPE (46676.php) could be investigated for privilege escalation.

---
