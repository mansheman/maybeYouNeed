2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Intelligence]] [[Threat Hunting]]
###### Prerequisites: [[Adversary Analysis and Correlation]]
# Threat Enrichment and CTI Integration

## Threat enrichment

Enrichment adds data about an indicator.

Common enrichment data:

- WHOIS
- passive DNS
- related hashes and malware families
- reputation data
- GeoIP and ASN context
- sandbox results

## Contextualization

Contextualization adds meaning around the enriched indicator, such as:

- campaign fit
- attack stage
- relevance to our sector, region, or stack

## CTI integration with SIEM and SOAR

CTI becomes operational when it changes:

- how detections are written
- how alerts are prioritized
- how response actions are triggered

Common uses:

- IP, URL, and hash matching
- ATT&CK-aligned rules
- automated blocking and enrichment

## Diagrams

![[Pics/ecthp/enrichment-context-workflow.svg|1200]]

![[Pics/ecthp/cti-integration-pipeline.svg|1200]]
