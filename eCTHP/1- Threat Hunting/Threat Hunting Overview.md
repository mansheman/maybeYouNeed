2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites:
# Threat Hunting Overview

Threat hunting is the proactive process of searching infrastructure for signs of malicious activity that routine monitoring did not detect.

## Typical data sources

- endpoint logs and telemetry
- network device logs
- authentication logs
- DNS and proxy logs
- firewall, IDS, and IPS records
- SIEM and data-lake events

## Proactive vs reactive

Reactive security operations:

- start from an alert
- rely on EDR, SOC, SIEM, IPS, AV, or other detections
- work well for known and noisy threats

Proactive threat hunting:

- starts from a hypothesis or assumption
- searches logs for evidence of compromise
- aims to identify threats that bypassed normal detection logic

## Assume breach

Threat hunting is built around an assume-breach mindset:

- do not assume the environment is clean
- assume a threat actor may already be present
- investigate systems and telemetry to validate or reject that assumption

## Diagram

![[Pics/ecthp/threat-hunting-spectrum.svg|1200]]
