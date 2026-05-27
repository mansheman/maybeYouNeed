2025-07-20 05:14

Status:

Tags: [[THM Web Fundementals]] [[tryhackme]] [[Tags/Command Injection|Command Injection]]
###### Prerequisites: 
# Detecting Command Injection
This vulnerability exists because applications often use functions in programming languages such as PHP, Python and NodeJS to pass data to and to make system calls on the machine’s operating system. For example, taking input from a field and searching for an entry into a file. Take this code snippet below as an example:

In this code snippet, the application takes data that a user enters in an input field named `$title` to search a directory for a song title. Let’s break this down into a few simple steps.

![](../../Pics/external/10db9612bdf00b0b.png)  

**1.** The application stores MP3 files in a directory contained on the operating system.

**2.** The user inputs the song title they wish to search for. The application stores this input into the `$title` variable.

**3.** The data within this `$title` variable is passed to the command `grep` to search a text file named _songtitle.__txt_ for the entry of whatever the user wishes to search for.

**4.** The output of this search of _songtitle.__txt_ will determine whether the application informs the user that the song exists or not.

Now, this sort of information would typically be stored in a database; however, this is just an example of where an application takes input from a user to interact with the application’s operating system.

An attacker could abuse this application by injecting their own commands for the application to execute. Rather than using `grep` to search for an entry in `songtitle.txt`, they could ask the application to read data from a more sensitive file.

Abusing applications in this way can be possible no matter the programming language the application uses. As long as the application processes and executes it, it can result in command injection. For example, this code snippet below is an application written in Python.

![](../../Pics/external/41d8a9bfc968b412.png)  

Note, you are not expected to understand the syntax behind these applications. However, for the sake of reason, I have outlined the steps of how this Python application works as well.

1. The "flask" package is used to set up a web server
2. A function that uses the "subprocess" package to execute a command on the device
3. We use a route in the webserver that will execute whatever is provided. For example, to execute `whoami`, we'd need to visit http://flaskapp.thm/whoami