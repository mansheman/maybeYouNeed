2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Threat Hunting Overview]]
# Hunt Phases and Hunt Types

## When to hunt

Threat hunting is not always triggered by a known threat or alert.

A team may hunt because:

- a hypothesis is worth testing
- a critical system is high-risk
- new intelligence suggests a realistic threat path

## Hunt phases

Threat hunting usually moves through:

1. preparation and data readiness
2. hypothesis creation
3. evidence collection and validation
4. escalation or detection improvement

## Hunt types

Structured hunts:

- begin with specific TTPs
- map well to ATT&CK and known adversary behavior

Unstructured hunts:

- begin with indicators of compromise
- often start from a hash, IP, domain, process, or user

Situational hunts:

- focus on specific systems or resources
- often center on critical assets, exposed services, or high-risk users
