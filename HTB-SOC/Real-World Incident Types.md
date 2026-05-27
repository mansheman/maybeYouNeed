# Real-World Incident Types
[[Tags/SOC]] [[Tags/HTB]] [[Tags/CaseStudy]]

Common attack vectors observed in the real world:

### Leaked Credentials
- **Colonial Pipeline (2021)**: Breached employee personal password for an inactive VPN account without MFA enabled.

### Default / Weak Credentials
- **Mirai Botnet (2016)**: Scanned for IoT devices using factory default credentials.
- **LogicMonitor (2023)**: Vendor issued weak default passwords to customer accounts.

### Outdated Software / Unpatched Systems
- **Equifax (2017)**: Exploited known Apache Struts vulnerability (CVE-2017-5638).
- **WannaCry (2017)**: Spread via SMB EternalBlue exploit on unpatched Windows systems.

### Rogue Employee / Insider Threat
- **Cash App / Block Inc. (2021)**: Former employee accessed personal info of millions of users.

### Phishing / Social Engineering
- **Twitter Hijacking (2020)**: Social engineering of employees to gain access to internal administrative tools.
- **U.S. Interior Department**: "Evil twin" fake Wi-Fi to steal credentials.

### Supply-Chain Attack
- **SolarWinds Orion (2020)**: Backdoor injected into software updates distributed to thousands of customers.

---
**Up:** [[HTB SOC Course MOC]]
**Related:** [[Incident Reporting]]
