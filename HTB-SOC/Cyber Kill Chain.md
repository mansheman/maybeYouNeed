# Cyber Kill Chain
[[Tags/SOC]] [[Tags/HTB]] [[Tags/KillChain]]

The Cyber Kill Chain describes the attack lifecycle, providing insights into an attacker's progress and access level.

### The Seven Stages
![[Pics/HTB-SOC/Cyber_kill_chain.png]]

1. **Recon (Reconnaissance)**: Choosing a target and gathering information (Active vs. Passive).
   ![[Pics/HTB-SOC/ir_recon.png]]
2. **Weaponize**: Developing malware and embedding it into an exploit or payload.
3. **Delivery**: Delivering the payload (phishing, malicious links, social engineering, USB).
4. **Exploitation**: Triggering the exploit to execute code on the target system.
5. **Installation**: Running a stager, dropper, backdoor, or rootkit for persistence.
6. **Command and Control (C2)**: Establishing remote access to the compromised machine.
7. **Action on Objectives**: Final goal (data exfiltration, ransomware, etc.).

**Note:** Adversaries often operate non-linearly, repeating stages (e.g., repeating Recon after Installation to move deeper).

---
**Up:** [[HTB SOC Course MOC]]
**Related:** [[MITRE ATT&CK Framework]]
