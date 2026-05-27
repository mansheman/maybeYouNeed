2026-04-17 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Threat Hunting]] [[Preparation]] [[Threat Hunting Overview]]
# Endpoint Threat Hunting

## Notes (MOC)

[[Endpoint Concepts and IOCs]]
[[Windows Execution, Persistence, and PowerShell]]
[[Sysmon for Endpoint Hunting]]
[[Endpoint Event IDs Cheat Sheet]]
[[Splunk for Endpoint Hunting]]
[[Splunk Search Cheat Sheet]]
[[ELK for Endpoint Hunting]]
[[ELK Search Cheat Sheet]]
[[Practical Splunk Threat Hunt Lab]]
[[Endpoint Hunting Recap and Next Steps]]

## Scope

These notes cover the full INE Endpoint Threat Hunting course.

## Why endpoint hunting matters

- Endpoints expose evidence of execution, persistence, credential abuse, and operator mistakes.
- Good endpoint hunts usually begin with either a specific hypothesis or a concrete trigger.
- The quality of the hunt depends on logging depth, especially PowerShell logs, Windows event logs, and Sysmon.

## Visual map

![[eCTHP/8- Endpoint Threat Hunting/endpoint-hunt-loop.svg|1000]]

![[eCTHP/8- Endpoint Threat Hunting/endpoint-persistence-map.svg|1000]]

![[eCTHP/8- Endpoint Threat Hunting/sysmon-visibility-map.svg|1000]]

![[eCTHP/8- Endpoint Threat Hunting/process-triage-lens.svg|1000]]

![[eCTHP/8- Endpoint Threat Hunting/splunk-hunt-workflow.svg|1000]]

![[eCTHP/8- Endpoint Threat Hunting/elk-hunt-workflow.svg|1000]]

## Source boundary

Source: INE Endpoint Threat Hunting PDF, slides 8 to 68, plus end-of-course recap slides.
