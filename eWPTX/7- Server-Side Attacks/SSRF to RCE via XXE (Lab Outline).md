2026-02-06 21:20

Status:

Tags:[[eWPTX]][[Tags/Server-Side Attacks|Server-Side Attacks]][[Tags/XML|XML]]

###### Prerequisites:
- [[eWPTX/4- XML/XML External Entities (XXE)|XML External Entities (XXE)]]
- [[eWPTX/7- Server-Side Attacks/Server-Side Request Forgery (SSRF)|Server-Side Request Forgery (SSRF)]]

# SSRF to RCE via XXE (Lab Outline)

## Goal
Document an XXE finding that enables **server-side data exfiltration** and **internal network interaction**, and then capture how it can be *chained* into SSRF-driven impact (sometimes escalating further depending on the internal service you reach).

This note is intentionally an **outline** (safe to share). Keep any step-by-step exploitation details, payloads, and exact commands in a **private** note.

## Lab Setup (What I Capture)
- Target app URL(s) and the exact endpoint that accepts XML
- Where XML is processed (request body, file upload, SOAP endpoint, etc.)
- Parser behaviors:
  - Does it accept `DOCTYPE`?
  - Does it expand entities?
  - Does it allow external entities?
  - Is the response reflected (non-blind) or not (blind)?

## High-Level Attack Chain (Conceptual)
### 1) Confirm XML parser behavior
- Start with a normal XML sample and confirm parsing success/failure conditions.
- Identify whether XML content is reflected back to you (helps determine if the issue is blind).

### 2) XXE signal
If the server:
- Accepts a `DOCTYPE`, and
- Expands entities,
then XXE may be possible depending on external entity resolution rules.

### 3) Use XXE as a **server-side capability**
Depending on the target, XXE may enable:
- **Local file disclosure** (e.g., configuration, environment hints)
- **Internal service discovery** (e.g., loopback-only services)
- **SSRF-style outbound requests** (the XML parser becomes your “HTTP client”)

### 4) SSRF pivot and follow-on
Once you can coerce server-side requests:
- Map internal hosts/services (what exists, what responds, what errors)
- Identify a reachable internal service with useful behavior (admin panels, debug endpoints, metadata services, etc.)
- Determine the *impact class*:
  - sensitive data exposure
  - internal-only functionality access
  - credential theft
  - in some environments, further escalation (depends heavily on what’s reachable)

## Evidence Checklist (Screenshots / Notes)
- The request/response proving XML parsing
- The request/response proving entity expansion (or other XXE indicator)
- Logs or side-channel proof if blind (DNS / timing / out-of-band)
- Clear “why impact is real” narrative (what internal thing was reached, what data/behavior was exposed)

## Risk Notes (How I Explain It)
- XXE/SSRF chains often break trust boundaries: **internet → app → internal network**
- The server typically has more access than the attacker:
  - internal DNS, loopback services, private IP ranges
  - cloud metadata endpoints (cloud-specific)

## Mitigation (What I Recommend)
- Configure XML parsers to:
  - disable DTD processing where possible
  - disable external entity resolution
  - enforce secure parser options (vendor-specific)
- Validate/limit accepted content types and schemas
- Network egress restrictions from app servers (deny-by-default)
- Add monitoring for unusual internal fetch patterns

## Private Lab Notes
- Link to your private walkthrough note here (kept out of Git).

