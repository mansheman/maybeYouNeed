2025-06-14 10:14

Status:

Tags: [[Metasploit]] [[Cyber Security 101]] [[tryhackme]]
###### Prerequisites: [[Introduction]]



# Metasploit Framework

## Metasploit Console

- **Command**: `msfconsole`
    
- The primary CLI interface to interact with Metasploit's modules and tools.
    
- Used to search, configure, and launch exploits and payloads.
    

---

## Core Concepts

### Vulnerability

- A weakness in the system’s code, configuration, or design that can be exploited.
    

### Exploit

- Code that takes advantage of a vulnerability to execute actions on the target.
    

### Payload

- The code that runs after an exploit is successful (e.g., opening a shell, downloading files).
    

---

## Module Types

### Auxiliary

- **Purpose**: Support operations (e.g., scanning, fuzzing, information gathering).
    
- **Examples**: `scanner/`, `spoof/`, `gather/`, `dos/`, `voip/`
    
- **Note**: Often used before or during an attack to collect info or disrupt services.
    

---

### Encoders

- **Purpose**: Obfuscate exploits and payloads to bypass basic antivirus detection.
    
- **Limitations**: Modern AVs use behavior analysis, so success is limited.
    
- **Examples**: `x86/`, `x64/`, `php/`, `ruby/`, `generic/`
    

---

### Evasion

- **Purpose**: Specifically designed to bypass security features like Windows Defender or AppLocker.
    
- **Examples**: `applocker_evasion_msbuild.rb`, `windows_defender_exe.rb`
    

---

### Exploits

- **Purpose**: Actively exploit vulnerabilities in target systems.
    
- **Organized by**: OS or platform (e.g., `windows/`, `linux/`, `osx/`)
    
- **Note**: Often used in combination with a payload to gain system access.
    

---

### NOPs (No Operation Instructions)

- **Purpose**: Padding used to align shellcode, increase reliability of exploits.
    
- **Common use**: Intel `NOP` = `0x90`
    
- **Categories**: `x86/`, `x64/`, `armle/`, `php/`
    

---

### Payloads

- **Purpose**: Run specific code on the target after exploitation.
    
- **Types**:
    
    - **Singles**: Self-contained (e.g., open calculator, add user)
        
    - **Stagers**: Establish connection with attacker system
        
    - **Stages**: Delivered by stager, larger payload with advanced features
        
    - **Adapters**: Modify payload format (e.g., wrap in PowerShell)
        
- **Naming Tip**:
    
    - `shell_reverse_tcp` → inline (single)
        
    - `shell/reverse_tcp` → staged payload
        

---

### Post Modules

- **Purpose**: Post-exploitation actions (gather credentials, pivot, escalate privileges).
    
- **Organized by**: OS or platform
    
- **Examples**: `windows/`, `linux/`, `networking/`, `multi/`
    

---

## Module Directory Path (Default)

bash

CopyEdit

`/opt/metasploit-framework/embedded/framework/modules/`

- Navigate here to manually explore Metasploit’s modules.
    
- Useful for creating or studying custom modules.