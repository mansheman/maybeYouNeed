# Insight Nexus Incident Scenario
[[Tags/SOC]] [[Tags/HTB]] [[Tags/CaseStudy]]

A recurring scenario used to understand challenges in incident handling.

### Victim: Insight Nexus
A global market research firm handling sensitive data. Targeted by two distinct threat groups simultaneously.

### The Network Flow
![[Pics/HTB-SOC/insights1.png]]
*Insight Nexus Network Architecture and Monitoring*

### Attack Path 1
1. **Initial Access**: Default `admin/admin` credentials on internet-facing ManageEngine ADManager Plus after an update.
2. **Reconnaissance**: Logged in, mapped users and machines.
3. **Persistence**: Created new privileged Active Directory accounts.
4. **Pivot**: Used a new account to access an external RDP service exposed by misconfiguration.
5. **Impact**: Deployed spyware via MSI package using Group Policy Objects (GPOs) across multiple endpoints.

---
**Up:** [[HTB SOC Course MOC]]
**Related:** [[Incident Reporting]]
