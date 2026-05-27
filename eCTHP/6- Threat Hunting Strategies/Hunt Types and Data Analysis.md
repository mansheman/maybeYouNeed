2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Hunt Preparation and Hypotheses]]
# Hunt Types and Data Analysis

## Structured hunts

- begin from attacker methods and TTPs
- search for symptoms of a known technique

## Unstructured hunts

- begin from indicators of compromise
- usually start with hashes, filenames, users, or IPs

## Situational hunts

- focus on specific systems or resources
- often center on critical assets, exposed services, or high-risk users

## Data analysis for threat hunting

Data analysis turns a hypothesis into validated evidence.

Key habits:

- compare related events across systems
- use baselines to separate normal from suspicious
- treat anomalies as leads, not proof
- refine the hypothesis as evidence changes

## Diagram

![[Pics/ecthp/hunt-data-analysis-funnel.svg|1200]]
