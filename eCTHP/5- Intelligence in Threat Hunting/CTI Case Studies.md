2026-04-18 00:00

Status:

Tags: [[eCTHP]] [[Threat Intelligence]] [[Threat Hunting]]
###### Prerequisites: [[CTI in Cyber Defense]]
# CTI Case Studies

## CTI Lifecycle Case Study - Healthcare Ransomware

### Scenario

The course walks through a January 2024 ransomware incident affecting a healthcare provider, with patient data encrypted and critical systems locked. The example is used to show how CTI should flow through the lifecycle.

### Lifecycle walkthrough

- planning and direction: the CISO, CTI lead, and incident commander define what must be known about IOCs, TTPs, and exposed infrastructure
- collection: gather internal logs, SIEM events, and EDR telemetry, then add OSINT, ISAC, vendor, and dark web reporting
- processing and exploitation: normalize timestamps, IPs, and domains, then enrich with GeoIP, WHOIS, and MITRE mappings
- analysis and production: determine what is happening, who is behind it, what the objective is, and how to stop it
- dissemination: push flash reports, briefings, or tickets to SOC, executives, IR, legal, and external partners
- feedback: ask whether the intelligence improved detection or response and identify gaps for the next cycle

### Typical roles by stage

- direction: CISO, IR lead, CTI lead
- collection: CTI analysts and threat hunters
- processing: automation engineers and intel technicians
- analysis: intelligence analysts and the hunt team
- dissemination: CTI lead and report writers
- feedback: all stakeholders

### Operational outcomes

- tactical: enrich SOC alerts and block malicious infrastructure
- operational: track campaigns and monitor adversary TTP evolution
- strategic: support budgeting and long-term security priorities

### Takeaways

- the intelligence cycle is iterative and flexible
- direction and feedback are critical bookends
- CTI should move quickly without sacrificing accuracy
- integrating CTI into detection and incident response makes teams more proactive

## SolarWinds Case Study

### Background

The SolarWinds Orion compromise embedded the SUNBURST backdoor into a trusted software update and affected thousands of organizations, including U.S. government entities.

### Core timeline

- September 2019: initial access
- March 2020: trojanized update deployed
- December 2020: discovery and public disclosure

### Important attacker characteristics

- linked to a Russian SVR-associated group
- stealthy espionage with strong operational security
- techniques included supply chain compromise, SAML token forgery, LOLBAS use, and HTTPS-based C2

### CTI lessons by level

- strategic: vendor trust and national security risk
- operational: track C2 infrastructure and campaign scope
- tactical: map TTPs to ATT&CK for detection work
- technical: share hashes, domains, URLs, and certificates

### Sharing lesson

This case showed why multi-party sharing matters. FireEye, Microsoft, CISA, and others exchanged intelligence through STIX/TAXII, repositories, and distribution controls.

### Key takeaway

Detection cannot rely on malware alone. Supply chain incidents require trust modeling, broader telemetry, and cross-organization sharing.

### Diagram

![[Pics/ecthp/solarwinds-timeline.svg|1200]]

## NotPetya Case Study

### Background

The June 2017 NotPetya event was first treated like ransomware, but it was effectively a destructive wiper delivered through the MeDoc supply chain in Ukraine.

### What made it important

- used EternalBlue and EternalRomance
- harvested credentials with Mimikatz
- moved laterally with PsExec
- caused large-scale business disruption

### Intelligence lesson

Early technical signs suggested ransomware, but the broader strategic picture pointed to state-linked disruption and economic damage.

### Why analysts got challenged

- early classification delayed the right response
- cross-sector sharing was not immediate enough
- too much focus on malware details hid the higher-level intent

### CTI lessons by level

- strategic: cyberwarfare and deterrence implications
- operational: impact analysis across many organizations
- tactical: propagation and lateral movement techniques
- technical: IoCs, exploit traces, and YARA rules

### Key takeaway

Attribution is not just about who did it. It is also about why it was done and what outcome the attacker wanted.

### Diagram

![[Pics/ecthp/notpetya-intent-vs-classification.svg|1200]]
