2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prerequisites: [[XML Injection - Overview]]
# XML Injection - Attack Types

## Common XML-related injection / parser abuse issues

|Attack type|What gets manipulated|Typical impact|
|---|---|---|
|XML data manipulation|XML values/structure|data tampering, auth bypass|
|XML Tag Injection|new/modified tags|privilege escalation, parsing issues|
|XML Attribute Injection|attributes|privilege escalation, data tampering|
|XPath Injection|XPath expressions used to query XML|unauthorized data access, auth bypass|
|XXE|external entities / DTD features|file disclosure, SSRF, DoS|
|Billion Laughs|recursive entities|DoS (resource exhaustion)|
|XML bombs|huge/deep XML|DoS|
|XInclude Injection|external includes|file inclusion, data disclosure|
|XML Schema poisoning|schema/validation manipulation|validation bypass, unexpected data|
