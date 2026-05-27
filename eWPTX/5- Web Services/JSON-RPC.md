2026-02-04 22:04

Status:

Tags: [[eWPTX]] [[Web Services]]
###### Prerequisites: [[Web Service Implementations]]
# JSON-RPC

## Overview

JSON-RPC is a remote procedure call (RPC) protocol encoded in JSON.

It is similar to XML-RPC but usually preferred because:

- messages are more human-readable
- payloads are often smaller

---

## Request object

A JSON-RPC request is a JSON object typically including:

- `method`: method name
- `params`: array/object of arguments
- `id`: request ID to match responses

Example:

```json
{
  "jsonrpc": "2.0",
  "method": "subtract",
  "params": [42, 23],
  "id": 1
}
```
