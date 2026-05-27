2026-04-17 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Splunk for Endpoint Hunting]]
# Splunk Search Cheat Sheet

Use this note for fast query building, not theory.

## Search skeleton

```spl
index=<index> source=<source> sourcetype=<type> <field>=<value> ... | <command>
```

## Core syntax

| Pattern | Meaning |
| --- | --- |
| `index=sysmon` | search a specific index |
| `source="WinEventLog:Security"` | scope to Windows Security log events |
| `sourcetype=XmlWinEventLog:Microsoft-Windows-Sysmon/Operational` | narrow to a specific source type |
| `EventCode=1` | filter on a field |
| `Image="*powershell*"` | wildcard field match |
| `A AND B` | both must be true |
| `A OR B` | either may be true |
| `(A OR B) AND C` | group conditions |
| `| table field1 field2` | keep only useful output fields |
| `| stats count by field` | summarize by field |
| `| sort - count` | descending sort |
| `| dedup field` | remove duplicate rows |
| `| where count > 5` | second-stage filtering |
| `| eval newfield=...` | create or rename fields |

## Commands worth memorizing

| Command | Good for |
| --- | --- |
| `table` | readable triage output |
| `stats` | counts, grouping, summaries |
| `sort` | ranking |
| `top` | most common values |
| `rare` | unusual values |
| `dedup` | unique hosts, users, hashes |
| `where` | post-processing filters |
| `eval` | derived fields and field cleanup |

## Source filters you should remember

| Source | Use |
| --- | --- |
| `source="WinEventLog:Security"` | logons, process creation, account changes, task and service creation |
| `source="WinEventLog:System"` | service installation and system-level events |
| `source="WinEventLog:Windows PowerShell"` | classic PowerShell logs |
| `source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"` | Sysmon telemetry |
| `source="XmlWinEventLog:Microsoft-Windows-TaskScheduler/Operational"` | scheduled task creation and modification |

## Triage patterns

### Show clean fields

```spl
index=sysmon EventCode=1 Image="*powershell*"
| table _time Computer User Image CommandLine ParentImage Hashes
```

### Scope Security log events first

```spl
index=wineventlog source="WinEventLog:Security" EventCode=4688
| table _time Computer SubjectUserName New_Process_Name Command_Line
```

### Count by host

```spl
index=sysmon EventCode=1 Image="*powershell*"
| stats count by Computer
| sort - count
```

### Find unique users

```spl
index=sysmon EventCode=1 Image="*powershell*"
| dedup User
| table User Computer Image CommandLine
```

### Find rare command lines

```spl
index=sysmon EventCode=1 Image="*powershell*"
| rare CommandLine
```

## Common hunt snippets

### Encoded PowerShell

```spl
index=sysmon EventCode=1 Image="*powershell*" CommandLine="*enc*"
| table _time Computer User CommandLine ParentImage
```

### PowerShell download behavior

```spl
index=sysmon EventCode=1 Image="*powershell*" (CommandLine="*Invoke-WebRequest*" OR CommandLine="*iwr*")
| table _time Computer User CommandLine
```

### Suspicious temp execution

```spl
index=sysmon EventCode=1 (CommandLine="*\\Temp\\*" OR Image="*\\Temp\\*")
| table _time Computer User Image CommandLine ParentImage
```

### Service creation

```spl
index=wineventlog (source="WinEventLog:Security" OR source="WinEventLog:System") (EventCode=4697 OR EventCode=7045)
| table _time Computer User Service_Name Service_File_Name
```

### Scheduled task creation

```spl
index=wineventlog source="WinEventLog:Security" (EventCode=4698 OR EventCode=4702)
| table _time Computer User TaskName TaskContent
```

### New user creation

```spl
index=wineventlog source="WinEventLog:Security" EventCode=4720
| table _time Computer SubjectUserName TargetUserName
```

### Log clearing

```spl
index=wineventlog source="WinEventLog:Security" EventCode=1102
| table _time Computer SubjectUserName
```

### Match a known hash

```spl
index=sysmon EventCode=1 Hashes="*<hash>*"
| table _time Computer User Image CommandLine Hashes
```

### DNS pivot from suspicious process activity

```spl
index=sysmon (EventCode=1 OR EventCode=22) Image="*powershell*"
| table _time Computer User Image CommandLine QueryName QueryStatus
```

## Query workflow

1. set the right time range
2. choose the right index
3. add `source` if you know the exact Windows log channel
4. add `sourcetype` if known
5. start narrow with fields and event codes
6. validate the raw events
7. only then summarize with `stats`, `top`, `rare`, or `dedup`

## Common mistakes

- forgetting the index
- searching the wrong time window
- forgetting `source="WinEventLog:Security"` for Security hunts
- using too many wildcards
- summarizing before checking raw events
- searching raw text when a fielded search is available

## Quick build formula

```spl
index=<where> source=<log channel> EventCode=<what happened> <field>=<suspect> | table <what you need to read>
```
