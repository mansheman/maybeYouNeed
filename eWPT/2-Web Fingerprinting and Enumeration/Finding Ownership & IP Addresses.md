2025-05-22 16:44

Status:

Tags: [[eWPT]]  [[Web Fingerprinting and Enumeration]] #Information_Gathering
###### Prerequisites:
[[Web Enumeration & Information Gathering]]


# WHOIS

## Definition
- A **query and response protocol**.
- Used to query databases that store registration information for internet resources, primarily:
    - **Domain names**
    - **IP address blocks**
- Reveals registered users or organizations associated with these resources.

## Purpose
- To identify **ownership information** for target domains and IP addresses.
- Key step in "Finding Website Ownership & IP Addresses."

## Methods for Lookups
- **Command Line Interface (CLI):** Via a `whois` client.
- **Web-Based Tools:** Numerous third-party websites provide WHOIS lookup capabilities.
- These methods access various databases to retrieve details.

## Significance in Web Enumeration
- Provides foundational ownership details crucial for the initial stages of information gathering.
- Helps map out a target's internet presence.
Okay, here are your notes on WHOIS commands and using the `host` command for reverse lookups, in an Obsidian-friendly format:


# WHOIS Commands & Reverse DNS with `host`

## Using the `whois` Command

The `whois` command is a command-line utility to query WHOIS databases for domain name and IP address registration information.

### Basic Syntax
```bash
whois [options] <domain_name_or_IP_address>
```

### Examples
-   **Query a domain name:**
    ```bash
    whois example.com
    ```
-   **Query an IP address:**
    ```bash
    whois 8.8.8.8
    ```
-   **Get help (common):**
    ```bash
    whois --help
    man whois
    ```

### Key Information to Look For
-   **Registrant Information:** Name, organization, address, email, phone (often masked by privacy services).
-   **Registrar:** The company that registered the domain.
-   **Registration Dates:** Creation, update, and expiration dates.
-   **Name Servers (NS Records):** Authoritative DNS servers for the domain. These can sometimes reveal IP addresses or lead to further DNS enumeration.
-   **Domain Status:** e.g., `clientTransferProhibited`, `ok`.
-   **Associated IP Blocks (for IP queries):** Organization owning the IP range, contact details (abuse, technical).

### Notes
-   Output format varies depending on the registrar and the TLD (Top-Level Domain).
-   Many registrars offer **WHOIS privacy services**, which will replace the actual owner's information with proxy details.
-   WHOIS servers may **rate-limit** queries from a single IP to prevent abuse.

---

## Reverse DNS Lookup with `host`

Once you identify an IP address (e.g., from a `whois` lookup on a domain, especially from Name Server details, or if you're directly investigating an IP), you can perform a reverse DNS lookup to find associated hostnames using the `host` command.

### Basic Syntax
```bash
host <IP_address>
```

### Example
Let's say a `whois` query for `example.com` or analysis of its Name Servers led you to an IP `93.184.216.34`.

```bash
host 93.184.216.34
```
**Expected Output (example):**
```
34.216.184.93.in-addr.arpa domain name pointer example.com.
```
This tells you that `93.184.216.34` points to `example.com`.

### Purpose of Reverse Lookup
-   **Verification:** Confirm if an IP is legitimately associated with a target domain.
-   **Discovery:** Potentially uncover other hostnames or services hosted on the same IP if the PTR (Pointer) record is configured.
-   **Mapping:** Helps build a more complete picture of the target's infrastructure.

### Notes on `host` Command
-   The `host` command can also perform forward DNS lookups (hostname to IP): `host example.com`
-   Not all IP addresses have a PTR record configured for reverse DNS. If not, the command might return a "not found" or similar message.
-   An IP can have multiple PTR records, or one PTR record that differs from the primary domain name initially queried.

---

## Workflow Example:

1.  **Initial Domain Query:**
    ```bash
    whois targetcompany.com
    ```
    *Look for Name Servers, IP addresses, or any hints in the registration data.*

2.  **Identify an IP:** Suppose you find an IP like `192.0.2.10` associated with one of their services or nameservers.

3.  **Reverse DNS Lookup on the IP:**
    ```bash
    host 192.0.2.10
    ```
    *This might reveal a specific server hostname like `mail.targetcompany.com` or `server1.targetcompany.net`.*

---
