# LSASS Access Exercise
[[Tags/SOC]] [[Tags/HTB]] [[Tags/MitreAttack]] [[Tags/LabNotes]]

Exercise identifying credential dumping via LSASS access in TheHive.

### Task
Check the alert with reference `67c202` (LSASS Access) in TheHive and provide the MITRE rule ID.

### Details
- **Credentials**: `htb-analyst` / `P3n#31337@LOG`
- **Alert Details**: LSASS process was accessed by `C:\Users\administrator.TECHRANGE\Downloads\mimikatz_trunk\x64\mimikatz.exe` with read permissions.
- **Indication**: Possible credential dump using Mimikatz.

![[Pics/HTB-SOC/Screenshot 2026-04-28 at 9.37.23 AM.png]]

![[Pics/HTB-SOC/Screenshot 2026-04-28 at 9.39.41 AM.png]]

---
**Up:** [[HTB SOC Course MOC]]
**Related:** [[TheHive Case Management]], [[MITRE ATT&CK Framework]]
