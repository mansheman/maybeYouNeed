2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Network IOCs and Triggers]]
# ARP, ICMP, and DHCP Hunting Patterns

## ARP

Suspicious signals:

- very high ARP broadcast volume
- gratuitous ARP replies
- the same MAC address appearing with different IPs

## ICMP

Suspicious signals:

- large numbers of echo requests
- replies without matching requests
- unusually large ICMP payloads

## DHCP

Suspicious signals:

- offers without corresponding requests
- out-of-order DHCP traffic
- rogue DHCP servers issuing offers

## Diagram

![[Pics/ecthp/arp-icmp-dhcp-abnormality.svg|1200]]
