2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[OAuth]]
###### Prerequisites: [[Token-Based Authentication]]
# Introduction to OAuth

## Overview

OAuth2 is the main web standard for **authorization** between services.

It allows third-party applications to access resources on a provider on behalf of a user, with user consent.

---

## OAuth components

- Resource Owner: typically the end-user
- Client: the app requesting access (relying party)
- Resource Server: API hosting protected resources
- Authorization Server: authenticates user and issues tokens (IdP)
- User Agent: browser/mobile app

---

## OAuth scopes

Scopes represent requested privileges/actions, e.g.:

- read
- write
- view_gallery
