2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Hunting]]
###### Prerequisites: [[Threat Hunting Overview]]
# Hunt Operations and Detection Gaps

## Threat hunting vs incident response

Threat hunting and incident response are closely related, but they are not the same activity.

Threat hunting:

- is a passive investigative activity
- focuses on finding hidden threats
- often starts without a confirmed incident

Incident response:

- is an active operational activity
- focuses on containment, eradication, and recovery
- starts when an incident is confirmed or strongly suspected

## How they work together

Threat hunters discover suspicious activity and build hypotheses.

Incident responders validate compromise details, contain the threat, and often provide new artifacts for future hunts.

## AV and EDR limitations

AV and EDR products usually depend on predefined detection logic such as:

- signatures
- file hashes
- filenames
- user-behavior baselines
- rules based on known tactics

If an attack does not match an existing rule:

- no alert may be generated
- no action may be taken
- the attacker may continue operating quietly

## Better defenses through hunting

Threat hunting improves defenses because it reveals:

- missing detections
- weak assumptions about normal activity
- blind spots in telemetry
- process gaps between hunting and response
