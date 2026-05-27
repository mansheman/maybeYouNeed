# MITRE ATT&CK Framework
[[Tags/SOC]] [[Tags/HTB]] [[Tags/MitreAttack]]

A granular, matrix-based knowledge base of adversary behavior (Windows, Linux, macOS, cloud, etc.).

### Framework Structure
![[Pics/HTB-SOC/mitreintro.png]]

- **Tactic**: High-level objective (e.g., Initial Access, Persistence).
- **Technique**: Specific method to achieve a tactic (e.g., T1105 Ingress Tool Transfer).
- **Sub-technique**: Specific implementation (e.g., T1003.001 LSASS Memory Dumping).

### Mapping Example
| Tactic | Technique | ID | Description |
| :--- | :--- | :--- | :--- |
| Initial Access | Exploit Public-Facing App | T1190 | Confluence CVE |
| Execution | PowerShell | T1059.001 | Payload download |
| Persistence | Windows Service | T1543.003 | Persistence |
| Credential Access | LSASS Dumping | T1003.001 | Extracted credentials |
| Lateral Movement | RDP | T1021.001 | Lateral movement |
| Impact | Data Encrypted | T1486 | LockBit ransomware |

---
**Up:** [[HTB SOC Course MOC]]
**Related:** [[Cyber Kill Chain]], [[Pyramid of Pain]]
