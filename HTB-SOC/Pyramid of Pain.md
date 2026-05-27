# Pyramid of Pain
[[Tags/SOC]] [[Tags/HTB]] [[Tags/MitreAttack]]

Illustrates the effort required for an adversary to change tactics when indicators are blocked.

![[Pics/HTB-SOC/ir_mitre.png]]

- **Hash Values (Trivial)**: Easily changed.
- **IP Addresses (Easy)**: Can quickly switch C2 servers.
- **Domain Names (Simple)**: Registering new domains is easy.
- **Network/Host Artifacts (Annoying)**: Registry keys, mutexes, etc.
- **Tools (Challenging)**: Forces attacker to use different tools.
- **TTPs (Tough)**: Disrupting core behaviors forces fundamental changes.

**Summary:**
- Hash/IP detections are **easy to evade**.
- Behavioral TTP detections (MITRE-based) are **hard to evade** and cause maximum pain to the attacker.

---
**Up:** [[HTB SOC Course MOC]]
**Related:** [[MITRE ATT&CK Framework]]
