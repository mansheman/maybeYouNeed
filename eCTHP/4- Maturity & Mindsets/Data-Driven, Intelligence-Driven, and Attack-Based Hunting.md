2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Threat Hunting Mindsets]]
# Data-Driven, Intelligence-Driven, and Attack-Based Hunting

## Intelligence-based hunting

Starts from intelligence such as:

- IPs
- domains
- file hashes
- filenames
- known attacker TTPs

## Data-driven hunting

Starts from internal telemetry that may indicate malicious activity, such as:

- lower-priority alerts
- suspicious aggregated patterns
- unusual authentication
- strange network connections

## Knowledge-based hunting

Relies on analyst understanding of:

- available logs and events
- network layout
- adversary TTPs

## Attack-based vs analytics-based

Attack-based hunting searches for evidence of a specific attack path or event.

Analytics-based hunting looks for suspicious or malicious patterns in systems, network, or data.

Mature teams usually combine both.
