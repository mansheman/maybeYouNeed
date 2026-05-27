2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Threat Hunting]] [[Preparation]]
# Hunt Preparation and Hypotheses

## Preparing to hunt strategically

Preparation determines hunting capability.

Before a hunt starts, you need:

- relevant intelligence
- the right logs
- enough time coverage
- access to the systems being investigated

Centralized logs make it easier to:

- search across assets
- compare related events
- pivot from one clue into broader activity

## Building hunt hypotheses

A hunt hypothesis is a testable assumption about malicious activity.

Good hypotheses:

- identify a specific behavior or activity to search for
- point to the data sources needed to test it
- stay narrow enough to remain testable

Helpful formula:

If an attacker is using `X`, then I should be able to observe `Y` in `Z` logs across `S` systems during `T` time.

## Diagrams

![[Pics/ecthp/prepare-to-hunt-cycle.svg|1200]]

![[Pics/ecthp/hypothesis-building-flow.svg|1200]]
