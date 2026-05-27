2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Network IOCs and Triggers]]
# Traffic Patterns and Unexpected Transfers

Most network hunting depends on knowing what normal traffic looks like.

Patterns to watch:

- unusual protocol usage
- larger-than-normal transfers
- smaller transfers that repeat over time
- traffic during off-hours
- regular beacon-like timing
- workstations talking to unusual destinations

Direction matters:

- internal to internal can suggest lateral movement
- internal to external can suggest exfiltration
- external to internal can suggest payload delivery or remote control

## Diagram

![[Pics/ecthp/traffic-patterns-directionality.svg|1200]]
