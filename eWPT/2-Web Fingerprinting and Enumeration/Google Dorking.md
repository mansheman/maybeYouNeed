2025-05-23 14:44

Status:

Tags: [[eWPT]]  [[Web Fingerprinting and Enumeration]] #Information_Gathering
###### Prerequisites:

[[Web Server Metafiles]]

# Google Dorking (Google Hacking)

Google Dorking, also known as Google Hacking, is an advanced search technique that uses specialized search operators (dorks) to find specific information on the internet that might not be readily apparent through regular searches. It's a powerful tool for reconnaissance, Open Source Intelligence (OSINT), and identifying potential security vulnerabilities.

**Ethical Considerations:**
Always use Google Dorking responsibly and ethically. Accessing systems or data without authorization is illegal. These techniques are for educational purposes, penetration testing with explicit permission, or legitimate research.

---

## Basic Site Operators

These operators help narrow down searches to specific websites or parts of websites.

### `site:example.com`
*   **Purpose:** Finds all indexed pages under `example.com`.
*   **Example:** If `example.com` has pages like `example.com/about` and `example.com/contact`, this dork will list them.

### `site:example.com inurl:example`
*   **Purpose:** Finds pages within `example.com` where the string "example" appears in the URL.
*   **Example:** Might find `example.com/example-page` or `example.com/blog/my-example-post`. This can be useful for finding specific sections or pages if "example" is part of a naming convention.

### `site:*.example.com inurl:example`
*   **Purpose:** Finds pages on `example.com` AND all its subdomains (e.g., `blog.example.com`, `dev.example.com`) where the string "example" appears in the URL. The `*` acts as a wildcard for subdomains.
*   **Example:** `site:*.example.com inurl:login` would find login pages across all subdomains of `example.com`.

---

## Advanced File & Content Operators

These operators help find specific content within pages or specific file types.

### `intitle:` Operator
Searches for a specific term within the HTML title of a webpage.

#### `site:*.example.com intitle:example`
*   **Purpose:** Finds pages on `example.com` or any of its subdomains where the word "example" appears in the page title.
*   **Example:** If `blog.example.com` has a page titled "My Example Post," this dork would find it.

### `filetype:` Operator
Restricts searches to specific file extensions.

#### `site:*.example.com filetype:example`
*   **Purpose:** Searches for files with the extension ".example" (e.g., `config.example`, `notes.example`) hosted on `example.com` or any of its subdomains.
*   **Note:** The usefulness of `filetype:example` is specific. More common and often sensitive filetypes include:
    *   `filetype:pdf` (PDF documents)
    *   `filetype:xls` or `filetype:xlsx` (Excel spreadsheets)
    *   `filetype:doc` or `filetype:docx` (Word documents)
    *   `filetype:txt` (Text files)
    *   `filetype:sql` (SQL dumps)
    *   `filetype:bak` (Backup files)
    *   `filetype:log` (Log files)
    *   `filetype:cfg` or `filetype:conf` (Configuration files)
    *   `filetype:env` (.env files often containing credentials)
*   **Combining:** `site:example.com filetype:pdf confidential` looks for PDF files on `example.com` that also contain the word "confidential".

---

## Vulnerability: Directory Listing

Web servers can sometimes be misconfigured to show a list of files and directories when a specific index file (like `index.html` or `index.php`) is not present in a directory. This is known as "Directory Listing."

### Dork: `intitle:"index of"`
*   **Purpose:** This dork specifically searches for pages with titles that commonly indicate an enabled directory listing. Web servers (like Apache) often generate pages with titles like "Index of /directoryname/" when directory listing is enabled.
*   **What it reveals:**
    *   The structure of directories on the server.
    *   Files that might not be directly linked from the website but are present on the server.
    *   Potentially sensitive files like configuration files, backups, source code, or internal documents.
*   **Why it's a vulnerability:**
    *   **Information Disclosure:** Attackers can gain insights into the web application's structure and potentially discover hidden or sensitive files.
    *   **Unintended Access:** Sensitive data not intended for public view might be exposed.

### Combining `intitle:"index of"` with `site:`
*   To target a specific website: `site:example.com intitle:"index of"`
*   To target subdomains as well: `site:*.example.com intitle:"index of"`

### Other variations for Directory Listings:
Sometimes directory listings might have slightly different titles. You can try variations like:
*   `intitle:"parent directory"`
*   `intext:"Index of /" "Last modified" "Parent Directory"` (looks for common phrases in directory listing pages)

---

## Finding Older Website Versions

Accessing older versions of a website can reveal information that has since been removed or changed.

### 1. Google Cache: `cache:example.com`
*   **Purpose:** Displays Google's cached version of `example.com`.
*   **Usage:** Simply prepend `cache:` to the URL you want to view.
*   **Limitations:** Usually provides only one recent cached version. Site owners can prevent caching.

### 2. Wayback Machine (archive.org)
*   **Website:** `https://archive.org/web/`
*   **Usage:** Go to the Wayback Machine, enter the URL (e.g., `example.com`), and browse the calendar for historical snapshots.
*   **Benefits:** Provides multiple historical snapshots over many years, revealing changes in content, design, and potentially exposed data from the past.

---

## Sensitive File Exposure Dorks

Google Dorks can find unintentionally exposed sensitive files. **Accessing these without permission is illegal and unethical.**

### `inurl:auth_user_file.txt`
*   **Purpose:** Searches for files named `auth_user_file.txt` in URLs.
*   **Significance:** This filename is sometimes used by certain web servers or applications to store usernames and (often hashed) passwords for HTTP Basic Authentication.
*   **Risk:** If found and readable, it could expose user credentials.

### `inurl:passwd.txt` OR `intitle:passwd.txt`
*   **Purpose:** Searches for files named `passwd.txt` either in the URL or in the page title.
*   **Significance:** While unlikely to be the system shadow file, it might be an accidental export of user lists or notes with credential-related information.
*   **Risk:** Could reveal usernames or passwords if misconfigured.

### `inurl:wp-config.bak`
*   **Purpose:** Searches for backup files of the WordPress configuration file (`wp-config.php`) accessible via URL.
*   **Significance:** The `wp-config.php` file contains crucial database connection details:
    *   Database name (`DB_NAME`)
    *   Database username (`DB_USER`)
    *   **Database password (`DB_PASSWORD`)**
    *   Database host (`DB_HOST`)
    *   Authentication Unique Keys and Salts.
*   **Risk:** If a backup (`.bak`, `.old`, `.txt`) of this file is web-accessible, it can often be downloaded as plain text, **revealing the MySQL (or other) database credentials**. This could lead to full database compromise.
    *   Also try variations like: `filetype:bak wp-config`, `filetype:txt wp-config "DB_PASSWORD"`

### Other Potentially Sensitive File Dorks:
*   `filetype:env "DB_PASSWORD"` (Looks for .env files with database passwords)
*   `filetype:sql "CREATE TABLE" "password"` (SQL dumps potentially containing table structures and data)
*   `filetype:log "password" "login"` (Log files that might erroneously store credentials)
*   `intext:"BEGIN RSA PRIVATE KEY"` (Exposed private SSH/TLS keys)
*   `inurl:.git/config` (Exposed Git repository configuration, potentially leaking repository information)

---

## Resources for Google Dorking

### Google Hacking Database (GHDB) - Exploit DB
The Google Hacking Database (GHDB) is a curated collection of Google dorks submitted by the security community. It categorizes dorks by the type of information or vulnerability they might uncover.
*   **Hosted by:** Exploit Database
*   **Link:** `https://www.exploit-db.com/google-hacking-database`
*   **Features:** Categorized dorks, search functionality, submission details, and explanations.
