2025-05-26 22:13

Status:

Tags: [[eWPT]] [[tryhackme]] [[tryhackme]]
###### Prerequisites:



# Windows Command Line Notes

## Prerequisites

Basic understanding of command-line navigation and syntax.

---

## Command Execution Context

- Commands are executed within directories listed in the system `Path` variable.
    

Check environment variables:

```cmd
set
```

Look for the `Path=` line. Example entries:

```cmd
C:\Windows\system32;
C:\Users\strategos\AppData\Local\Microsoft\WindowsApps;
```

---

## System Information Commands

View Windows version:

```cmd
ver
```

Example output:

```cmd
Microsoft Windows [Version 10.0.17763.1821]
```

Detailed system information:

```cmd
systeminfo
```

Includes hostname, OS version, manufacturer, build type, etc.

---

## Useful Output Handling

Paginate long command outputs:

```cmd
driverquery | more
```

Press space to scroll, CTRL + C to stop.

---

## Helpful Commands

```cmd
help
cls
```

---

# Network Configuration & Troubleshooting

## View Network Configuration

Basic network info:

```cmd
ipconfig
```

Detailed network configuration:

```cmd
ipconfig /all
```

Includes DNS servers, DHCP status, MAC address, etc.

---

## Troubleshooting Connectivity

Ping remote host:

```cmd
ping example.com
```

Trace route to remote host:

```cmd
tracert example.com
```

DNS resolution:

```cmd
nslookup example.com
```

Query a specific DNS server:

```cmd
nslookup example.com 1.1.1.1
```

---

## Viewing Network Connections

Active connections and listening ports:

```cmd
netstat
```

Extended detailed view:

```cmd
netstat -abon
```

Options:

- `-a` : show all connections and listening ports
    
- `-b` : show the executable responsible for each connection
    
- `-o` : show PID of owning process
    
- `-n` : numeric addresses (no DNS lookup)
    

Example output shows `sshd.exe` listening on port 22 with PID information.

Help for options:

```cmd
netstat -h
```

---

# Working with Directories

|Command|Description|
|---|---|
|`cd`|Show current directory|
|`cd folder_name`|Change to subdirectory|
|`cd ..`|Move up one directory level|
|`dir`|List contents of current directory|
|`dir /a`|Include hidden/system files|
|`dir /s`|Include contents of all subdirectories|
|`tree`|Visual display of directory tree|
|`mkdir folder`|Create new directory|
|`rmdir folder`|Remove empty directory|

---

# Working with Files

|Command|Description|
|---|---|
|`type filename.txt`|Display file content|
|`more filename.txt`|Paginate long file content|
|`copy source.txt dest.txt`|Copy a file|
|`move file.txt folder\`|Move a file to another directory|
|`del filename.txt`|Delete a file|
|`erase filename.txt`|Alternative to `del`|
|`copy *.ext folder\`|Copy all files of specific type|

---

## Example Workflow

```cmd
cd Documents
mkdir testfolder
cd testfolder
copy C:\example\test.txt .
type test.txt
move test.txt ..
cd ..
erase test.txt
```

---

# Viewing Running Processes

Show all running processes:

```cmd
tasklist
```

Details provided:

- Image Name
    
- PID (Process ID)
    
- Session Name
    
- Memory Usage
    

---

## Filtering by Image Name

```cmd
tasklist /FI "imagename eq <process_name.exe>"
```

Example:

```cmd
tasklist /FI "imagename eq sshd.exe"
```

Displays only processes matching `sshd.exe`.

---

# Killing Processes

Kill process by PID:

```cmd
taskkill /PID <pid_number>
```

Example:

```cmd
taskkill /PID 2712
```

Force kill a process by PID:

```cmd
taskkill /PID <pid_number> /F
```

Kill all instances of a process by name:

```cmd
taskkill /IM <process_name.exe> /F
```

Example:

```cmd
taskkill /IM sshd.exe /F
```

Help for task filtering and options:

```cmd
tasklist /?
taskkill /?
```

---

## Example Workflow for Process Management

```cmd
tasklist /FI "imagename eq notepad.exe"
taskkill /IM notepad.exe /F
```

Lists all running `notepad.exe` instances and forcibly terminates them.

---

# Essential Commands Recap

```cmd
cd
dir
tree
mkdir
rmdir
type
more
copy
move
del
ipconfig
ping
tracert
nslookup
netstat
tasklist
taskkill
```

---

# Additional Useful Commands

```cmd
chkdsk
driverquery
sfc /scannow
```

---

# Pro Tips

- Use `/F` with `taskkill` to force process termination.
    
- Use wildcards (`*.txt`) to target multiple files.
    
- Combine commands using pipes (`|`) for advanced workflows.
    
- Use `/??` on most commands to see help and syntax:
    

```cmd
copy /?
```

---

This is the **FULL**, clean, properly structured version with **NO parts missing**.

Let me know if you want me to break it into smaller notes or prepare PowerShell/advanced Windows notes next.