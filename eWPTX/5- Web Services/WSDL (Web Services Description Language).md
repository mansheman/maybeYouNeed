2026-02-04 22:04

Status:

Tags: [[eWPTX]] [[Web Services]] [[XML]]
###### Prerequisites: [[SOAP]]
# WSDL (Web Services Description Language)

## Overview

WSDL is an XML-based language used to describe the functionality and interface of a web service.

It acts as a **contract** between provider and consumer, specifying:

- operations
- input/output structures
- bindings (how messages are sent)
- endpoints (where the service is)

WSDL is commonly used with SOAP services.

---

## WSDL versions

- WSDL 1.1
- WSDL 2.0 (newer, but 1.1 still widely used)

---

## Abstract vs concrete definitions

- **Abstract**: what the service does (operations + messages)
- **Concrete**: how/where it is offered (bindings + endpoints)

---

## Components (high level)

- **Types**: defines data types (usually XSD schemas)
- **Message**: message structures (WSDL 1.1)
- **PortType**: operations supported (WSDL 1.1)
- **Binding**: protocol/encoding details
- **Service**: endpoint information (URL)

WSDL 2.0 note:

- uses **Interface** instead of `portType`
- does not use `message` the same way; it points more directly to schema elements
