# Preparation Stage - Policies and Tools
[[Tags/SOC]] [[Tags/HTB]] [[Tags/Preparation]] [[Tags/Policy]] [[Tags/Forensics]]

Detailed requirements for documentation and equipment during the preparation phase.

### Essential Documentation
- **Contact Lists**: Internal IH team, legal, IT support, law enforcement, ISPs, and external IR vendors.
- **Network & Asset Data**: Network diagrams, system baselines (golden images), and asset management databases.
- **Access Management**: Pre-defined high-privilege accounts for emergency use (enabled only during incidents with immediate password resets).
- **Procedures**: IR plans, information sharing policies, and forensic cheat sheets.
- **Financial Mandate**: Ability to bypass slow procurement for urgent tool purchases (e.g., up to $500).

### Software & Hardware Tools
- **Forensic Workstations**: Isolated laptops for malware testing and data analysis (AV disabled).
- **Analysis Tools**: Software for disk imaging, memory capture, log analysis, and network capture.
- **Physical Equipment**: Write blockers, hard drives, network cables/switches, power cables, and hardware repair kits.
- **The "Jump Bag"**: A pre-packed bag containing all necessary tools for immediate deployment.

### Strategic Security
- **Independent Systems**: Documentation and communication systems must be **independent** of the main organization infrastructure. 
- **Assumption of Compromise**: Assume the adversary controls the domain and can read internal emails; use out-of-band communication.

---
**Up:** [[HTB SOC Course MOC]]
**Related:** [[Preparation Stage - Prerequisites]]
