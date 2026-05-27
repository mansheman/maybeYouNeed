2026-04-17 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Endpoint Concepts and IOCs]] [[Preparation]]
# Splunk for Endpoint Hunting

Splunk is the SIEM layer that makes endpoint hunting practical at scale.

## What Splunk gives the hunter

- log and data analysis in one place
- collection from multiple sources
- aggregation and correlation across host data
- real-time analysis and alerting
- dashboards and visualizations
- raw-event searching through SPL
- add-on applications that extend detection and parsing

## Why it matters in endpoint hunting

Endpoint logs are useful only if they can be searched across many systems and time windows.

Splunk helps the hunter move from one host artifact to cross-host visibility.

## Good mental model

Windows and Sysmon tell you what happened on the host.

Splunk helps you ask:

- where else did this happen?
- when did it start?
- which hosts and users are involved?
- what related events cluster around it?

## Architecture for hunting

The simple Splunk architecture in the course has:

- forwarder: collects and sends logs from endpoints or other sources
- indexer: stores and indexes incoming data
- search head: where the hunter runs searches and reviews results

If a query returns nothing, do not assume the behavior never happened.

First confirm:

- the correct index
- the correct source
- the correct sourcetype
- the correct time range
- that the relevant logs were actually forwarded

## SPL basics

Core building blocks:

- `index`
- `source`
- `sourcetype`
- field filters
- pipes `|`
- commands
- boolean logic and grouping

High-value commands:

- `table`
- `stats`
- `sort`
- `top`
- `dedup`
- `where`
- `eval`

Useful habits:

- narrow with `index=` first
- add `source=` when you know the exact Windows log channel
- add `sourcetype=` when you know it
- prefer fielded searches over broad raw-text searches
- validate raw events before summarizing

Important Windows example:

`source="WinEventLog:Security"`

That scopes the search to Windows Security log events instead of searching more broadly than needed.

## Building hunt queries

Good Splunk queries start with a good hunt hypothesis.

Do not begin with syntax.

Begin with the attacker behavior you expect to be visible.

Example hypothesis from the course:

Attackers are running suspicious scripts from temporary folders.

Possible search directions:

- `.ps1` files executed inside the hunt time window
- non-executable files launched from `C:\\Windows\\Temp` or `C:\\Temp`
- script files created in temp folders

Better workflow:

1. write the hypothesis in plain language
2. identify the relevant logs
3. map the behavior to fields and event codes
4. run a narrow first search
5. pivot based on what comes back

Strong query purposes:

- new user creation
- brute-force attempts
- unexpected outbound network connections
- suspicious PowerShell
- persistence through tasks, autoruns, or services

## Search tips

- ensure the time range is appropriate
- format results into tables for readability
- use `stats`, `count`, `sort`, `top`, and `rare`
- use `where` to filter results further
- use `eval` to rename fields or create calculated fields
- remember the index first
- include `source` when you know the exact Windows log
- include `sourcetype` when known to reduce scope
- search fields directly when possible
- use wildcards only when needed because they can hurt performance

Common failure modes:

- wrong time range
- wrong index
- forgetting the correct `source`, especially Windows Security logs
- wrong sourcetype
- too-broad wildcard searches
- jumping into stats before validating raw events

## Threat hunting workflow in Splunk

Threat hunting in Splunk is proactive and hypothesis-driven.

Strong workflow:

1. define the hypothesis
2. pick the relevant data sources and indexes
3. run an initial narrow search
4. inspect the raw events
5. pivot into summary views, related hosts, and adjacent behaviors
6. turn confirmed patterns into future detections or playbooks

Splunk does not replace endpoint knowledge.

It amplifies endpoint knowledge by letting you search it across the environment.

## PowerShell hunt example

The course example focuses on this hypothesis:

An attacker used PowerShell to download malware through obfuscated or encoded commands.

Step 1: identify relevant logs

- Windows Event Logs
- Sysmon
- network data such as DNS and firewall logs

Step 2: execute the first query

```spl
index=sysmon EventCode=1 Image="*powershell*" CommandLine="*enc*"
```

This looks for Sysmon process-creation events where PowerShell appears with an encoded command.

Step 3: inspect and pivot

- exact command line
- host name
- user
- hashes
- related file names
- related network activity

Additional queries from the course:

```spl
index=sysmon EventCode=1 Image="*powershell*" CommandLine="*update.ps1*"
index=wineventlog EventCode=4688 New_Process_Name="*powershell*" Command_Line="*update.ps1*"
index=sysmon EventCode=1 CommandLine="*update.ps1*" | table _time, Computer, Image, CommandLine, Hashes
index=sysmon EventCode=1 Hashes="*<hash>*" | table _time, Computer, Image, CommandLine, User, Hashes
```

The first query is not the conclusion.

It is only the door into the rest of the evidence.

## Diagram

![[eCTHP/8- Endpoint Threat Hunting/splunk-hunt-workflow.svg|1000]]

## Practical lab

[[Practical Splunk Threat Hunt Lab]]

## Quick refs

[[Splunk Search Cheat Sheet]]
