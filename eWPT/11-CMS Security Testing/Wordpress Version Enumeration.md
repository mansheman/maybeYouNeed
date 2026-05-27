2025-08-27 14:49

Status:

Tags: [[eWPT]] [[File & Resource Attacks]]
###### Prerequisites: 
# WordPress Version Enumeration

## 1. Enumeration Methodology

The goal of enumeration is to collect information about the target WordPress instance in order to identify its version, themes, plugins, and potential vulnerabilities. A structured approach includes:

- **Information Gathering & Enumeration**
    
    - Perform **port scanning** (identify web server, database, and open services).
        
    - Identify the **WordPress version** running on the site.
        
    - Enumerate installed **themes and plugins** along with their versions.
        
    - Perform **file & directory enumeration** to uncover hidden or sensitive files (e.g., backups, configs, readme files).
        

---

## 2. Version Enumeration Techniques

### **Manual Techniques**

- **Meta Generator Tag**  
    Inspect the HTML source for the `<meta name="generator" content="WordPress X.X.X">` tag.
    
- **readme.html / license.txt**  
    WordPress installations often include `readme.html` or `license.txt` files that disclose the version.
    
- **HTTP Response Headers**  
    Some servers disclose WordPress version or PHP details in headers like `X-Powered-By`.
    
- **Login Page (`/wp-login.php`)**  
    The WordPress login page sometimes reveals the version in the page source.
    
- **REST API**  
    Access the JSON API (e.g., `http://example.com/wp-json/`) to check for version fields.
    
- **JS / CSS File Versions**  
    Many theme and plugin assets (e.g., `style.css?ver=5.8.1`) leak version information.
    
- **Changelog Files**  
    Files like `changelog.txt` or `readme.txt` inside plugin/theme directories may reveal update history and version.
    

---

### **Automated Techniques**

- **WPScan**  
    A dedicated WordPress security scanner that can enumerate WordPress version, users, plugins, and themes.
    
- **CMSmap**  
    Automated CMS scanner supporting WordPress, Joomla, Drupal, etc.
    
- **Other Tools**
    
    - `wpscan --url http://target.com`
        
    - `whatweb` (CMS and version detection)
        
    - `nmap --script http-wordpress-enum`
        

Automated tools speed up the process and often include built-in vulnerability checks.