2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[ELK for Endpoint Hunting]]
# ELK Search Cheat Sheet

Use this as the fast-reference note for the ELK section.

## KQL basics

KQL is field-based and simple.

Core ideas from the course:

- `:` works like equals
- use boolean logic with `AND`, `OR`, `NOT`
- use wildcards when needed
- prefer field-based searches over broad full-text searching
- save commonly used queries

### KQL patterns

```text
field:value
field:"exact phrase"
field:(value1 OR value2)
field:*partial*
```

### Example KQL-style query directions

```text
winlog.event_data.Image:"*reg.exe*" and winlog.event_data.CommandLine:"*save*" and winlog.event_data.CommandLine:"*\\HKLM\\SAM*"
```

```text
winlog.event_data.CommandLine:("*procdump.exe*" or "*mimikatz.exe*")
```

```text
event.code:4624 and message:"*ntlm*"
```

## EQL basics

EQL is better for event correlation and sequences.

Simple shape from the course:

```text
event_type where condition
```

Example:

```text
process where process.name == "powershell.exe"
```

Useful operators:

- `==`
- `!=`
- `<`
- `>`
- `and`
- `or`
- `not`

## EQL superpower

EQL is especially good when the hunt depends on order, for example:

- first a suspicious process starts
- then a logon occurs
- then a remote execution utility appears

That is the ELK equivalent of hunting the attack chain rather than a single event.

## Common ELK hunt targets from the course

- suspicious account logins
- suspicious process behavior
- unexpected outbound network connections
- suspicious PowerShell
- persistence through tasks or services

## Quick examples from the course

```text
winlog.event_data.CommandLine:("*wmic.exe*" or "*psexec.exe*" or "*smbexec.py*") and NOT winlog.event_data.User:"*SYSTEM*"
```

```text
winlog.event_data.ParentImage:"*wmiprvse.exe*" and winlog.event_data.CommandLine:("*cmd.exe*" or "*powershell.exe*")
```

## Practical rule

Use KQL when you need fast filtering.

Use EQL when the hunt depends on correlation and sequence.
