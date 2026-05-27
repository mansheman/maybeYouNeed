# Incident Handling Lifecycle
[[Tags/SOC]] [[Tags/HTB]] [[Tags/IncidentHandling]]

The Incident Handling Process defines an organization's capability to prepare, detect, and respond to malicious events. It is cyclic and non-linear, meaning next steps change as new evidence is discovered.

### NIST Incident Response Life Cycle
1. **Preparation**
2. **Detection and Analysis**
3. **Containment, Eradication, and Recovery**
4. **Post-Incident Activity**

![[Pics/HTB-SOC/ir-lifecycle.png]]
*Incident Response Life Cycle Model*

### Key Principles
- **Time Allocation**: Most time is spent in **Preparation** and **Detection & Analysis**. Resources must always remain in these stages even during active response to prevent disruption of monitoring.
- **No Skipping Steps**: Complete each step fully before moving on. *Example*: Do not start eradication on 5 machines if 10 are known to be infected; this tips off the attacker.
- **Relationship to Cyber Kill Chain**: While they don't map 1:1, the process allows handlers to predict and anticipate the attacker's next moves.

### Main Activities
#### 1. Investigation
- Discover "Patient Zero".
- Create an ongoing incident timeline.
- Determine tools and malware used.
- Document compromised systems and attacker actions.

#### 2. Recovery
- Create and implement a recovery plan.
- Resume normal business operations.

#### 3. Post-Incident
- Issue a report detailing cause and cost.
- Perform "Lessons Learned" to prevent recurrence.

---
**Up:** [[HTB SOC Course MOC]]
**Related:** [[Incident Handling Definition]], [[Cyber Kill Chain]]
