2025-08-21 21:32

Status:

Tags: [[eWPT]] [[File & Resource Attacks]]
###### Prerequisites: 
# Bypassing Extension filters file upload

so we have an ngnix server if we try to upload a php shell
![[Pics/Screenshot 2025-08-21 at 9.21.46 PM.png]]

it only accepts jpg and png and bmp

so i uploaded shell.php so the app know content type is application/php but intercepted in burp and made the extenstion to .jpg and uploaded it and opend it

![[Pics/Screenshot 2025-08-21 at 9.27.06 PM.png]]


but an  image cant do nothing so i told the server to execute it as php since its nginx i just added /shell.php and it worked

![[Pics/Screenshot 2025-08-21 at 9.27.32 PM.png]]

and i just placed my parameter cmd and navigated to get the flag

![[Pics/Screenshot 2025-08-21 at 8.44.09 PM 2.png]]
