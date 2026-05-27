2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Logging and Analysis Infrastructure]]
# Log Scope, Baselines, and Access

## Log preparation

Ensure logs are collected from:

- endpoints
- servers
- network devices
- authentication systems
- cloud platforms
- applications

Logs must also be retained long enough to support hunts.

## What to log

- account and group activity
- firewall and internal network traffic
- file creation, modification, and deletion
- email flow and filtering events
- Windows PowerShell, system, application, and Sysmon logs
- security-relevant application logs
- public-facing web, email, gateway, and API activity

## Baselines

A baseline is a documented measure of normal behavior and configuration.

Examples:

- expected file hashes
- expected open network ports
- approved accounts
- expected privileges

## Access

Hunters need enough access to validate hypotheses, including:

- SIEM search access
- EDR console access
- endpoint and server logs
- authentication logs
- network and packet-capture access
- cloud and application log access
