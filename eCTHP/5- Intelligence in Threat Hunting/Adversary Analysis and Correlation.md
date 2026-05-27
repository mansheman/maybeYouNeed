2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Intelligence]] [[Threat Hunting]]
###### Prerequisites: [[Threat Intelligence Levels]]
# Adversary Analysis and Correlation

## Adversary analysis with ATT&CK

ATT&CK helps analysts map observed behavior to known adversary tradecraft instead of treating each indicator as an isolated fact.

It is useful for:

- mapping threat-group behavior
- comparing behavior across incidents
- hunting for activity using TTPs
- prioritizing detection engineering

## IoC correlation

Correlation links indicators across time, tools, and sources so isolated data points become an intelligence picture.

Common methods:

- exact matching
- infrastructure pivoting
- fuzzy matching
- time-based reconstruction
- TTP and campaign linking

## Why it matters

- validates threats using multiple sources
- improves context and actor understanding
- finds low-signal activity
- reveals campaign overlap and likely intent

## Diagrams

![[Pics/ecthp/attack-mapping-workflow.svg|1200]]

![[Pics/ecthp/ioc-correlation-methods.svg|1200]]
