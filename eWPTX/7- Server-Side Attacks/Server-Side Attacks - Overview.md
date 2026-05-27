2026-02-05 09:06

Status:

Tags: [[eWPTX]] [[Server-Side Attacks]]
###### Prerequisites: [[Modern Web Application Architecture]]
# Server-Side Attacks - Overview

## What “server-side” means

Server-side refers to processes, logic, and infrastructure running on the server (not the browser).

This includes:

- request processing
- business logic execution
- communication with databases/APIs
- integration with internal services

---

## What server-side attacks target

Server-side attacks exploit weaknesses in the server infrastructure/components (web server, app server, APIs, internal services).

Potential impact:

- unauthorized access
- data breaches
- internal network access
- full system compromise

Course focus note:

- here we focus mainly on application/business layer issues (SQLi/NoSQLi already covered elsewhere).

---

## How to recognize server-side vulns (criteria)

- **Execution context**: vulnerability triggers in server-side code
- **Scope**: affects infra/config/inter-component comms
- **Component interaction**: abuses how server talks to other services
- **Trust assumptions**: internal trust relationships abused (SSRF, deserialization)
- **Server resources**: filesystem/CPU/memory/network capabilities exploited
