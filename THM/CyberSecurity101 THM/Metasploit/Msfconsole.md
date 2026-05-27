2025-06-14 10:16

Status:

Tags: [[Metasploit]] [[Cyber Security 101]] [[tryhackme]]
###### Prerequisites:


## Introduction to Metasploit Console

The **console** (`msfconsole`) is the main interface for interacting with the Metasploit Framework. To launch it, use:

```bash
msfconsole
```

Once launched, you'll see a banner and the prompt will change to something like:

```
msf6 >
```

This indicates you're in the Metasploit console environment.

---

## Using Shell Commands in `msfconsole`

Many Linux shell commands can be executed within `msfconsole`. For example:

```bash
msf6 > ls
[*] exec: ls
```

```bash
msf6 > ping -c 1 8.8.8.8
[*] exec: ping -c 1 8.8.8.8
```

However, not all shell features work. For example, output redirection is not supported:

```bash
msf6 > help > help.txt
[-] No such command
```

---

## Built-In Features and Commands

### Help System

The `help` command displays usage info for commands:

```bash
msf6 > help set
```

### Command History

You can review previous commands with:

```bash
msf6 > history
```

### Tab Completion

Metasploit supports tab completion. For example, typing `he` then pressing Tab auto-completes to `help`.

---

## Contextual Modules

Modules are used in context. For example, using the EternalBlue exploit:

```bash
msf6 > use exploit/windows/smb/ms17_010_eternalblue
```

Now the prompt shows:

```
msf6 exploit(windows/smb/ms17_010_eternalblue) >
```

Each module has its own context. If you switch to another module, your previously set options (unless global) won't carry over.

You can list module-specific options with:

```bash
msf6 exploit(...) > show options
```

You can exit the current module context with:

```bash
msf6 exploit(...) > back
```

---

## Searching and Information

### Search for Modules

To find modules related to a vulnerability:

```bash
msf6 > search ms17-010
```

### View Module Information

Within a module context:

```bash
msf6 exploit(...) > info
```

Or directly from the base prompt:

```bash
msf6 > info exploit/windows/smb/ms17_010_eternalblue
```

This displays the author, description, references, and usage info for the module.

---

## Using and Viewing Payloads

To list compatible payloads with a module:

```bash
msf6 exploit(...) > show payloads
```

When you select a module with `use`, it may auto-select a default payload. You can change it with:

```bash
msf6 exploit(...) > set PAYLOAD windows/x64/meterpreter/reverse_tcp
```

---

## Summary

- Launch Metasploit with `msfconsole`
    
- Use `help`, `history`, and tab completion for assistance
    
- Use `use`, `show options`, `set`, `run`, `back`, and `info` to interact with modules
    
- `search` and `show payloads` are essential for finding appropriate tools
    
- Each module context is separate, so settings don’t carry between them unless global
    

§