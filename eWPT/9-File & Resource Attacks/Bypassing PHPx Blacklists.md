2025-08-23 18:55

Status:

Tags: [[eWPT]] [[File & Resource Attacks]]
###### Prerequisites: 
# Bypassing PHPx Blacklists

so first when you upload a php shell it gets uploaded normally
![[Pics/Screenshot 2025-08-23 at 6.50.26 PM.png]]
 but when you open the file you uploaded it treats it like data
 ![[Pics/Screenshot 2025-08-23 at 6.50.40 PM.png]]

so it doesn't work
when you use burp suite you notice the webserver is using php
![[Pics/Screenshot 2025-08-23 at 6.52.12 PM.png]]



so what i did is i minipulated the extension of the file so it bypasses the filters
![[Pics/Screenshot 2025-08-23 at 6.53.18 PM.png]]





And Bingo it worked
![[Pics/Screenshot 2025-08-23 at 6.53.49 PM.png]]



and i proceeded to get the flag from /var/www/html/flag
![[Pics/Screenshot 2025-08-23 at 6.54.24 PM.png]]
