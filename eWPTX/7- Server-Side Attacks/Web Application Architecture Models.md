2026-02-05 09:06

Status:

Tags: [[eWPTX]] [[Server-Side Attacks]]
###### Prerequisites: [[Modern Web Application Architecture]]
# Web Application Architecture Models

|Model|Description|Best use case|
|---|---|---|
|Layered|Distinct layers (presentation, business, data)|Traditional/monolithic apps|
|Microservices|Small independent services via APIs|Large scalable systems|
|SOA|Reusable centralized services + ESB|Enterprise integrations|
|Event-driven|Async triggers on events/actions|Real-time (e-commerce/IoT)|
|Serverless|Event-triggered functions (FaaS)|Lightweight workloads|
|Monolithic|All components in one unit|Small apps / early stage|
|Distributed|Spread across multiple nodes/systems|High scalability + fault tolerance|

---

## Layered architecture (quick)

Common layers:

- presentation (client-side)
- business/application (app server)
- data access (database)

Communication is usually top-down.
