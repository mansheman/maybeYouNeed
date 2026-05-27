2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites:
# Logging and Analysis Infrastructure

Threat hunting depends on data being collected before an investigation begins.

## Collection and aggregation

Logs should be ingested into a central location so hunters can search across systems.

Common sources:

- endpoint telemetry
- server logs
- authentication logs
- network device logs
- cloud logs
- application logs

## Common tools

- SIEM platforms such as Splunk or the ELK stack
- packet capture and analysis tools such as Wireshark
- EDR platforms for richer endpoint telemetry
- threat-intelligence feeds and reports

Each tool adds different visibility:

- SIEM centralizes and searches events
- packet capture inspects network behavior
- EDR adds endpoint-level detail
- threat intelligence adds context and known indicators

## Infrastructure logging

Infrastructure logging preserves telemetry, enables cross-system correlation, and makes normal versus abnormal activity easier to detect.
