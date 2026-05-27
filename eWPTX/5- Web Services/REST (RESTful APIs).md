2026-02-04 22:04

Status:

Tags: [[eWPTX]] [[Web Services]]
###### Prerequisites: [[Web Service Implementations]]
# REST (RESTful APIs)

## Overview

REST (Representational State Transfer) is an architectural style for designing networked applications.

It is not a protocol itself; it’s a set of constraints that guide how APIs/services use HTTP.

REST services often use JSON or XML, but any format can be used.

---

## HTTP methods (common mapping)

|Method|Action|Examples|
|---|---|---|
|GET|Retrieve resource / list collection|`/book` (list), `/book/1` (view)|
|PUT|Replace / update (or create if absent)|`/book/1` (replace)|
|POST|Create new resource|`/book` (create)|
|DELETE|Delete resource|`/book/1` (delete)|
