2026-04-17 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Endpoint Concepts and IOCs]] [[Logging and Analysis Infrastructure]]
# Windows Execution, Persistence, and PowerShell

Most endpoint hunts on Windows come down to execution, persistence, and what happened immediately before and after.

## Process triage

Process hunting is about validating whether a running executable fits the normal Windows execution chain.

Fields that matter first:

- process name
- PID
- executable path
- parent process
- signature
- command line

Fast triage questions:

- Did the expected parent process spawn it?
- Is it running from the expected location?
- Is the name spelled correctly?
- Is it signed by Microsoft when it should be?
- Does the command line fit a real user or admin action?
- Did it create children, files, registry changes, or network connections right after start?

Commonly abused Windows processes:

- `smss.exe`
- `csrss.exe`
- `winlogon.exe`
- `wininit.exe`
- `services.exe`
- `lsass.exe`
- `svchost.exe`
- `taskhostw.exe`
- `explorer.exe`

High-risk relationships:

- Office spawning `cmd.exe`, `powershell.exe`, or `wscript.exe`
- `wscript.exe` or `cscript.exe` spawning networking or archive tools
- a browser spawning `powershell.exe` unexpectedly
- `rundll32.exe` or `regsvr32.exe` with strange command lines
- `svchost.exe` starting from a non-system path

The process name alone is weak.

The parent-child relationship plus path plus command line is what usually exposes abuse.

## Service persistence

Services are a persistence favorite because they can start automatically and often run with elevated privileges.

How attackers abuse services:

- create a new service with `sc.exe create`
- create a service with `New-Service`
- use WMI or third-party tools
- write directly under `HKLM\\SYSTEM\\CurrentControlSet\\Services\\`
- replace an existing service executable
- change recovery actions so a failure launches another program

Indicators:

- `sc.exe create` in logs
- `New-Service` in logs
- registry changes under `HKLM\\SYSTEM\\CurrentControlSet\\Services\\`
- Sysmon `12`, `13`, `14`
- Windows `4697` and `7045`
- unexpected users creating or modifying services

## Scheduled task persistence

Scheduled tasks blend into legitimate administration and can trigger in many ways.

Common triggers:

- date or time
- recurring intervals
- user logon
- system startup
- workstation lock or unlock

Abuse patterns:

- `schtasks /create`
- `New-ScheduledTask`
- GUI-based creation in Task Scheduler
- `schtasks /change`
- `Set-ScheduledTask`
- encoded PowerShell inside the task action

Detection indicators:

- Windows `4698`, `4700`, `4702`
- Task Scheduler operational `106`, `140`

What matters most:

- the action
- the trigger
- the author
- the run account
- the executable or script being launched

## PowerShell logging

PowerShell is one of the most important Windows data sources for endpoint hunting because attackers use it for download, execution, discovery, and lateral movement.

Events to remember:

- `4103`: module logging
- `4104`: script block logging

Terms to recognize quickly:

- `Invoke-Expression` or `iex`
- `Invoke-WebRequest` or `iwr`
- `EncodedCommand`

Why these matter:

- `4103` helps show what cmdlets were used
- `4104` often shows the real script content, even when the command line is short
- encoded commands commonly hide intent on the command line

Hunt questions:

- Did PowerShell launch from a suspicious parent process?
- Did it download content or reach out to a remote host?
- Did it create a task, service, user, or registry persistence key?
- Did it run shortly before suspicious network or file events?

If PowerShell logging is missing, note the blind spot explicitly.

Missing telemetry is itself part of the hunt story.

## Diagram

![[eCTHP/8- Endpoint Threat Hunting/endpoint-persistence-map.svg|1000]]

![[eCTHP/8- Endpoint Threat Hunting/process-triage-lens.svg|1000]]
