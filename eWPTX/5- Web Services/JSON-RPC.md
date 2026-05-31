2026-02-04 22:04

Status:

Tags: [[eWPTX]] [[Web Services]]
###### Prasyarat: [[Web Service Implementations]]
# JSON-RPC

## Gambaran singkat

JSON-RPC adalah protokol remote procedure call (RPC) yang di-encode dalam JSON.

Mirip XML-RPC, tapi sering lebih disukai karena:

- pesannya lebih mudah dibaca
- payload biasanya lebih kecil

---

## Objek request

Request JSON-RPC biasanya berupa object JSON yang berisi:

- `method`: nama method
- `params`: argumen (array/object)
- `id`: request ID untuk mencocokkan response

Contoh:

```json
{
  "jsonrpc": "2.0",
  "method": "subtract",
  "params": [42, 23],
  "id": 1
}
```
