2026-04-10 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Logging and Analysis Infrastructure]]
# Event IDs

## Why event IDs matter

Event IDs are one of the fastest ways to pivot during a hunt.

For Windows environments, Sysmon plus native Security logs usually provide the most useful signals.

## High-value Sysmon event IDs

- 1: Process Create
- 3: Network Connection
- 5: Process Terminated
- 7: Image Loaded
- 8: CreateRemoteThread
- 10: Process Access
- 11: File Create
- 12: Registry key created or deleted
- 13: Registry value set
- 14: Registry key/value renamed
- 15: File stream created
- 22: DNS Query
- 23: File deleted
- 25: Process Tampering

## Registry-related events

Registry changes are especially useful for persistence and defense evasion hunting.

Important IDs:

- Sysmon 12 for registry object create/delete
- Sysmon 13 for registry value changes
- Sysmon 14 for registry rename activity
- Windows Security 4657 for registry value modification when registry auditing is enabled

## Process and thread events

Process execution and injection activity often show up in:

- Sysmon 1 for process creation
- Sysmon 8 for remote thread creation
- Sysmon 10 for process access
- Windows Security 4688 for process creation

## Other important Windows Security events

- 4624: successful logon
- 4625: failed logon
- 4648: explicit credentials used
- 4672: special privileges assigned to a logon
- 4697: service installed
- 4719: audit policy changed
- 4720: user account created
- 4728 / 4732: user added to a group
- 4738: user account changed
- 4768: Kerberos TGT requested
- 4769: Kerberos service ticket requested
- 4771: Kerberos pre-authentication failed
- 7045: service installed in the system
- 1102: audit log cleared

## What to prioritize first

If you are hunting for hidden activity, the first events to check are usually:

- process creation
- network connections
- registry changes
- remote thread creation
- explicit credentials and logon activity
- service installation
