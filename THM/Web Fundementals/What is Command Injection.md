2025-07-20 03:00

Status:

Tags: [[THM Web Fundementals]] [[tryhackme]] [[Tags/Command Injection|Command Injection]]
###### Prerequisites: 
# What is Command Injection
To begin with, let’s first understand what command injection is. Command injection is the abuse of an application's behaviour to execute commands on the operating system, using the same privileges that the application on a device is running with. For example, achieving command injection on a web server running as a user named `joe` will execute commands under this `joe` user - and therefore obtain any permissions that `joe` has.

Command injection is also often known as “Remote Code Execution” (RCE) because of the ability to remotely execute code within an application. These vulnerabilities are often the most lucrative to an attacker because it means that the attacker can directly interact with the vulnerable system. For example, an attacker may read system or user files, data, and things of that nature.  

For example, being able to abuse an application to perform the command `whoami` to list what user account the application is running will be an example of command injection.