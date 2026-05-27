2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Endpoint Concepts and IOCs]] [[Logging and Analysis Infrastructure]]
# ELK for Endpoint Hunting

ELK is another SIEM-style stack for endpoint hunting, similar in purpose to Splunk but built around Elasticsearch, Logstash, and Kibana.

## What ELK is

The course frames ELK as three main components working together:

- Elasticsearch for search and analytics
- Logstash for data processing and transformation
- Kibana for dashboards and visualization

It often also includes Beats such as:

- Winlogbeat
- Filebeat

Those are used to ship logs into the stack.

## Why hunters care

ELK gives the same broad value proposition as Splunk:

- centralized log search
- cross-host visibility
- fast pivots between artifacts
- historical analysis
- dashboards and visualizations

## Architecture

Simple flow:

- Winlogbeat or Filebeat ships data
- Logstash parses and transforms it
- Elasticsearch stores and indexes it
- Kibana is where the hunter searches and visualizes

## Search languages in ELK

ELK supports multiple search languages.

### KQL

KQL is best for:

- faster and simpler queries
- field-based matching
- ad hoc filtering in Kibana

### EQL

EQL is best for:

- event correlation
- multi-step searches
- describing sequences such as “this happened, then that happened”

## Hunting workflow

1. identify relevant logs
2. choose the right language for the problem
3. start with a narrow field-based query
4. inspect the results
5. pivot into related processes, logons, or movement

## ELK hunt example from the course

Hypothesis:

An attacker dumped the SAM database, stole password hashes, and then used them in a Pass-the-Hash-style follow-on action.

Relevant logs:

- Windows Security logs
- Sysmon

Initial query direction:

- look for `reg.exe`
- look for `save`
- look for command lines touching `HKLM\\SAM`

The important lesson is the same as in Splunk:

the first query is only the entry point, not the conclusion.

## Diagram

![[eCTHP/8- Endpoint Threat Hunting/elk-hunt-workflow.svg|1000]]
