2026-04-17 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Event IDs]] [[Sysmon for Endpoint Hunting]]
# Endpoint Event IDs Cheat Sheet

Use this as the fast lookup version.

## First-pass priority

- execution: `Sysmon 1`, `Windows 4688`
- network and DNS: `Sysmon 3`, `Sysmon 22`
- persistence: `Sysmon 11`, `12`, `13`, `14`, `Windows 4697`, `7045`, `4698`, `4702`
- credential and injection-adjacent behavior: `Sysmon 8`, `10`, `Windows 4624`, `4625`, `4648`, `4672`
- account changes: `Windows 4720`, `4728`, `4732`, `4738`
- log tampering: `Windows 1102`, `Windows 4719`

## Sysmon quick reference

| ID | Meaning | Hunt use |
| --- | --- | --- |
| `1` | Process creation | parent-child chain, command line, execution |
| `2` | File creation time changed | timestomping clues |
| `3` | Network connection | outbound beacons, staging, unusual destinations |
| `6` | Driver loaded | kernel abuse or suspicious drivers |
| `8` | CreateRemoteThread | injection-like behavior |
| `9` | Raw disk access | low-level disk interaction |
| `10` | Process access | LSASS targeting, credential theft, process inspection |
| `11` | File created or overwritten | dropped payloads and staging |
| `12` | Registry object create/delete | persistence or config changes |
| `13` | Registry value set | Run keys, service values, defense changes |
| `14` | Registry rename | stealthy persistence updates |
| `19` | WMI event filter | WMI persistence and execution |
| `20` | WMI event consumer | WMI persistence chain |
| `21` | WMI event binding | WMI persistence chain |
| `22` | DNS query | process-to-domain correlation |
| `23` | File deleted | deleted payloads, cleanup attempts |
| `25` | Process tampering | hollowing/tampering indicators |

## Windows Security and System quick reference

| ID              | Meaning                           | Hunt use                                                |
| --------------- | --------------------------------- | ------------------------------------------------------- |
| `4624`          | Successful logon                  | unusual source, type, timing, privileged access         |
| `4625`          | Failed logon                      | brute force and password spraying                       |
| `4648`          | Explicit credentials used         | lateral movement and alternate credential use           |
| `4657`          | Registry value modified           | persistence and config changes when auditing is enabled |
| `4672`          | Special privileges assigned       | privileged logon context                                |
| `4688`          | Process creation                  | execution when Sysmon is absent or for cross-checking   |
| `4697`          | Service installed                 | persistence via service creation                        |
| `4698`          | Scheduled task created            | persistence or delayed execution                        |
| `4700`          | Scheduled task enabled            | re-enabled task abuse                                   |
| `4701`          | Scheduled task disabled           | stealth or cleanup                                      |
| `4702`          | Scheduled task updated            | modified persistence                                    |
| `4719`          | Audit policy changed              | logging suppression or tampering                        |
| `4720`          | User account created              | persistence through account creation                    |
| `4728` / `4732` | User added to group               | privilege escalation or persistence                     |
| `4738`          | User account changed              | suspicious account modification                         |
| `4768`          | Kerberos TGT requested            | authentication flow and abnormal account use            |
| `4769`          | Kerberos service ticket requested | service access and Kerberoasting context                |
| `4771`          | Kerberos pre-auth failed          | bad password activity and spray attempts                |
| `7045`          | Service installed                 | service-based persistence in system log                 |
| `1102`          | Audit log cleared                 | evidence destruction                                    |

## Task Scheduler operational channel

| ID | Meaning | Hunt use |
| --- | --- | --- |
| `106` | Task created | GUI or operational visibility for new tasks |
| `140` | Task updated | modified tasks and persistence changes |

## Fast hunt clusters

- suspicious PowerShell: `1`, `4688`, `4103`, `4104`, `3`, `22`
- malicious service: `4697`, `7045`, `12`, `13`, `14`, `1`
- malicious scheduled task: `4698`, `4700`, `4702`, `106`, `140`, `1`
- account abuse: `4624`, `4625`, `4648`, `4672`, `4720`, `4728`, `4732`, `4738`
- defense evasion: `4719`, `1102`, `25`

## Memory shortcut

If you blank during a hunt, start with:

`1`, `3`, `10`, `11`, `12`, `13`, `14`, `22`, `4688`, `4697`, `4698`, `4720`, `7045`, `1102`
