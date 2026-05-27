2026-04-17 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Logging and Analysis Infrastructure]] [[Event IDs]]
# Sysmon for Endpoint Hunting

Sysmon is one of the highest-value endpoint telemetry sources for Windows hunting.

## What it is

- part of the Sysinternals suite
- a Windows service and device driver
- adds logging that Windows does not provide by default
- writes its telemetry into the Windows event log

## Why hunters care

Sysmon improves visibility into the exact places attackers live:

- process execution
- hashes of executed files
- DLL and driver loading
- network connections
- file creation and overwrite
- registry changes
- DNS queries
- process tampering and cross-process access

## What makes it especially useful

- logs full command lines
- includes both current and parent process
- records file hashes
- can record source process, IP, port, and hostname for network activity

## Operational rule

Sysmon is most valuable when its logs are forwarded off-host into your central analysis platform.

Local-only visibility is fragile.

## High-value Sysmon events

- `1`: process creation with extended details
- `2`: file creation time changed
- `3`: network connection
- `6`: driver loaded
- `8`: remote thread created in another process
- `9`: raw disk access via `\\\\.\\`
- `10`: one process opened another process
- `11`: file created or overwritten
- `12`, `13`, `14`: registry object create-delete, value set, and rename activity
- `19`, `20`, `21`: WMI events
- `22`: DNS query
- `23`: file deleted and archived under `C:\\sysmon`
- `25`: process tampering detected

## What to prioritize first in a real hunt

- `1` for execution
- `3` and `22` for outbound behavior and name resolution
- `10` for credential theft or injection-adjacent activity
- `11` to spot dropped payloads
- `12`, `13`, `14` for persistence
- `19`, `20`, `21` for WMI-based execution or persistence
- `25` for process tampering

## A useful way to memorize them

- execution: `1`
- communication: `3`, `22`
- persistence and system changes: `11`, `12`, `13`, `14`, `19`, `20`, `21`
- process abuse: `8`, `10`, `25`
- lower-level visibility: `6`, `9`

## Important caveat

Sysmon is only as good as its configuration.

If a field or event type is missing, do not assume the behavior did not happen.

Sometimes the right conclusion is that the host has a telemetry gap.

## Shortcut

If you only remember one cluster, remember:

`1`, `3`, `10`, `11`, `12`, `13`, `14`, `22`, `25`

That cluster covers most of the first-pass endpoint hunt surface.

## Diagram

![[eCTHP/8- Endpoint Threat Hunting/sysmon-visibility-map.svg|1000]]
