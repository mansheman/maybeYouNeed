2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Network IOCs and Triggers]]
# TCP Hunting Patterns

Normal TCP normally follows the three-way handshake:

- SYN
- SYN/ACK
- ACK

Suspicious TCP patterns:

- excessive SYN packets
- unusual TCP flag combinations
- one host talking to many ports on the same target
- one host probing many different systems

These patterns often point to:

- port scanning
- service discovery
- stealthy reconnaissance

## Diagram

![[Pics/ecthp/tcp-normal-vs-suspicious.svg|1200]]
