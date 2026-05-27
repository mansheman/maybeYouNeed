2026-02-05 09:06

Status:

Tags: [[eWPTX]] [[Server-Side Attacks]]
###### Prerequisites: 
# Modern Web Application Architecture

When interacting with a web application through a browser, it may feel like a simple one-dimensional system.

Reality: modern web apps are layered systems with multiple interconnected components (web servers, app servers, APIs, databases, CDNs, queues, etc.).

---

## Why pentesters care

Understanding architecture helps you:

- map components and data flows
- identify layer-specific vulnerabilities
- understand how issues propagate across layers
- estimate realistic impact and mitigation

In server-side attacks, architecture matters because vulnerabilities like SSRF and insecure deserialization often exploit:

- internal trust relationships
- network reachability
- component interactions

---

## What web apps often consist of

- **Application servers**: business logic + dynamic requests
- **Database servers**: data storage/retrieval (usually not directly internet-facing)
- **Performance mechanisms**: CDNs, caches, load balancers

Often the domain points to intermediaries (LB/CDN) that hide origin servers.

![[Pics/server-side/web-app-architecture-layers.svg]]
