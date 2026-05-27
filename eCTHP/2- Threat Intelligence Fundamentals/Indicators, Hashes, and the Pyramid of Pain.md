2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Intelligence]]
###### Prerequisites:
# Indicators, Hashes, and the Pyramid of Pain

## Indicators of compromise

Common IOC examples:

- file hashes
- IP addresses
- domain names
- filenames
- suspicious strings
- anomalous behavior

## Hashes

A hash is a digital fingerprint for a file.

Common algorithms:

- MD5
- SHA-256
- SHA-512

Hashes are useful for validation and lookup, but easy for attackers to change by modifying the file.

## Pyramid of Pain

The Pyramid of Pain shows how difficult it is for an attacker to adapt when a defender detects different indicator types.

Lower levels:

- hashes
- IPs
- domains

Higher levels:

- tools
- TTPs

## Diagram

![[Pics/ecthp/pyramid-of-pain.svg|1200]]
