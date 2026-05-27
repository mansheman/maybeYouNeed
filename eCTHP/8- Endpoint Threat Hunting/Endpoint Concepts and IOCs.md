2026-04-17 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]] [[Threat Intelligence]]
###### Prerequisites: [[Threat Hunting]] [[Threat Hunting Overview]] [[Hunt Preparation and Hypotheses]]
# Endpoint Concepts and IOCs

Endpoint threat hunting is the practice of testing whether attacker activity is visible on systems through logs, artifacts, and behavior.

## Where hunts usually start

- a trigger from threat intelligence, incident response, or the SOC
- a concrete IOC such as a hash, filename, registry path, user, or IP
- a TTP-based hypothesis such as attackers creating scheduled tasks for persistence

## IOC hunts vs TTP hunts

- IOC hunt: start with a known artifact and search for related evidence
- TTP hunt: start with expected attacker behavior and search for traces of that behavior

## What separates hunting from alert triage

Alert triage starts with a system-generated signal and tries to confirm whether it is real.

Threat hunting starts with a human question and proactively looks for activity that may not have triggered an alert yet.

## Core workflow

1. define the trigger or hypothesis
2. identify which endpoint artifacts should contain evidence
3. search the relevant logs across the right time window
4. pivot from one artifact to related artifacts
5. decide whether the activity is benign, suspicious, or confirmed malicious

## What a good hunt produces

- one or more suspicious artifacts
- a chain of related evidence
- a decision about impact and confidence
- a detection idea that can be operationalized later

## What weak hunts look like

- searching one IOC and stopping
- treating a filename as proof
- ignoring host baseline and normal admin behavior
- failing to pivot into process, user, persistence, and network context

## Main endpoint IOC categories

### File-based IOCs

- filename
- file hash such as `MD5` or `SHA256`
- file path
- file size

How to think about them:

- filename is weak by itself because it is easy to rename
- hash is stronger, but attackers can still recompile or modify the file
- path helps when malware hides in unusual directories or mimics trusted locations
- size is useful for clustering payloads and staging artifacts

### Windows system-based IOCs

- registry keys
- scheduled tasks
- Windows services
- process execution
- account and logon events
- PowerShell logs

Examples:

- `reg add`
- `HKLM\\Software\\Microsoft\\Windows\\CurrentVersion\\Run`
- `schtasks /create`
- `sc.exe create`
- Windows `4624`, `4625`, `1102`
- PowerShell `4103`, `4104`

### Behavioral IOCs

- unexplained changes in system activity
- deviations in firewall or security settings
- unexpected installed software
- phishing-related artifacts
- new local or domain user creation
- suspicious downloads via PowerShell or utilities such as `certutil.exe`

Behavior matters because attackers can change names and hashes faster than they can avoid leaving execution and persistence traces.

## Why baselines matter

Most endpoint hunting gets easier once you know what normal looks like.

Useful baseline targets:

- common parent-child process chains
- standard software paths
- normal PowerShell usage
- expected scheduled tasks and services
- expected admin accounts and logon times
- expected outbound connections and DNS patterns

Without a baseline, "unusual" becomes subjective and noisy.

## Pivot checklist

When you find one suspicious artifact, pull the next layer of evidence:

1. identify the process or user connected to it
2. find the parent process and command line
3. check persistence tied to the same executable or user
4. check network, DNS, or download behavior around the same time
5. expand to other hosts for the same hash, path, service, task, or account

Examples:

- hash to path, host, parent process, first seen, last seen
- process to parent, child, signer, command line, network
- service to image path, creator, startup type, account context
- PowerShell command to script block, parent process, downloaded file, destination IP

## Data sources that matter most

- Windows Security logs
- Windows System and Application logs
- PowerShell logging, especially `4103` and `4104`
- Sysmon
- centralized SIEM data after collection and aggregation

## Fast mental model

If the attacker did something on the endpoint, ask:

- what process executed it
- what persistence mechanism kept it alive
- what log should show it
- what child artifact should appear next

## Diagram

![[eCTHP/8- Endpoint Threat Hunting/endpoint-hunt-loop.svg|1000]]
