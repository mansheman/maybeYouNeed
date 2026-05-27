2025-08-27 14:50

Status:

Tags: [[eWPT]] [[File & Resource Attacks]]
###### Prerequisites: 
# Wordpress Adrotate Lab
## so this is our lab lets start enumrate
![[Pics/Screenshot 2025-08-27 at 2.51.03 PM.png]]

## First lets enumrate manually by looking at the source code

![[Pics/Screenshot 2025-08-27 at 2.52.20 PM.png]]


## As you can see i found that its using wordpress 4.4

![[Pics/Screenshot 2025-08-27 at 2.55.09 PM.png]]
## Also by looking at the response we fount the php version and the server is apache  and its version and its running on ubuntu


# Automated Using WPScan

```bash
wpscan --url https://twl1ukoy4taqvovu5vx2arfmy.eu-central-7.attackdefensecloudlabs.com/
```

```python
──(ab0she㉿kali)-[~]
└─$ wpscan --url https://twl1ukoy4taqvovu5vx2arfmy.eu-central-7.attackdefensecloudlabs.com/
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.28
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: https://twl1ukoy4taqvovu5vx2arfmy.eu-central-7.attackdefensecloudlabs.com/ [172.104.136.144]
[+] Started: Wed Aug 27 07:37:18 2025

Interesting Finding(s):

[+] Headers
 | Interesting Entries:
 |  - server: Apache/2.4.7 (Ubuntu)
 |  - x-powered-by: PHP/5.5.9-1ubuntu4.25
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: https://twl1ukoy4taqvovu5vx2arfmy.eu-central-7.attackdefensecloudlabs.com/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: https://twl1ukoy4taqvovu5vx2arfmy.eu-central-7.attackdefensecloudlabs.com/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: https://twl1ukoy4taqvovu5vx2arfmy.eu-central-7.attackdefensecloudlabs.com/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: https://twl1ukoy4taqvovu5vx2arfmy.eu-central-7.attackdefensecloudlabs.com/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.4 identified (Insecure, released on 2015-12-09).
 | Found By: Rss Generator (Passive Detection)
 |  - https://twl1ukoy4taqvovu5vx2arfmy.eu-central-7.attackdefensecloudlabs.com/?feed=rss2, <generator>https://wordpress.org/?v=4.4</generator>
 |  - https://twl1ukoy4taqvovu5vx2arfmy.eu-central-7.attackdefensecloudlabs.com/?feed=comments-rss2, <generator>https://wordpress.org/?v=4.4</generator>

[+] WordPress theme in use: twentysixteen
 | Location: https://twl1ukoy4taqvovu5vx2arfmy.eu-central-7.attackdefensecloudlabs.com/wp-content/themes/twentysixteen/
 | Last Updated: 2025-08-05T00:00:00.000Z
 | Readme: https://twl1ukoy4taqvovu5vx2arfmy.eu-central-7.attackdefensecloudlabs.com/wp-content/themes/twentysixteen/readme.txt
 | [!] The version is out of date, the latest version is 3.6
 | Style URL: https://twl1ukoy4taqvovu5vx2arfmy.eu-central-7.attackdefensecloudlabs.com/wp-content/themes/twentysixteen/style.css?ver=4.4
 | Style Name: Twenty Sixteen
 | Style URI: https://wordpress.org/themes/twentysixteen/
 | Description: Twenty Sixteen is a modernized take on an ever-popular WordPress layout — the horizontal masthead wi...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.0 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - https://twl1ukoy4taqvovu5vx2arfmy.eu-central-7.attackdefensecloudlabs.com/wp-content/themes/twentysixteen/style.css?ver=4.4, Match: 'Version: 1.0'

[+] Enumerating All Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] adrotate
 | Location: https://twl1ukoy4taqvovu5vx2arfmy.eu-central-7.attackdefensecloudlabs.com/wp-content/plugins/adrotate/
 | Last Updated: 2025-08-22T23:45:00.000Z
 | [!] The version is out of date, the latest version is 5.15.1
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 3.9.4 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - https://twl1ukoy4taqvovu5vx2arfmy.eu-central-7.attackdefensecloudlabs.com/wp-content/plugins/adrotate/readme.txt

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:06 <================================================================================================> (137 / 137) 100.00% Time: 00:00:06

[i] No Config Backups Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Wed Aug 27 07:37:31 2025
[+] Requests Done: 172
[+] Cached Requests: 5
[+] Data Sent: 60.251 KB
[+] Data Received: 207.889 KB
[+] Memory used: 287.777 MB
[+] Elapsed time: 00:00:12
                                                                                                                                                                               
┌──(ab0she㉿kali)-[~]
└─$ 
                                                                                                                                                                               
┌──(ab0she㉿kali)-[~]
└─$ 

```
