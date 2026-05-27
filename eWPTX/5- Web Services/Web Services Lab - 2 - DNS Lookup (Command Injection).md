2026-02-04 23:32

Status:

Tags: [[eWPTX]] [[Web Services]]
###### Prerequisites: [[Web Services]] [[SOAP]]
# Web Services Lab - 2 - DNS Lookup (Command Injection)

## Source

- [[Web Services Lab]]

---
## Performing command injection attacks against the DNS Lookup web service.

**Step 14:** Open the DNS Lookup web service.

Visit **Web Services** > **SOAP** > **Command Injection** > **DNS Lookup**:

![[Pics/INE/14_b5afa313a1.png]]

That would take you to the following web page:

![[Pics/INE/14_1_3206d32edd.png]]

**Step 15:** Check the WSDL file.

Click on the WSDL link to view the WSDL file for the provided web service:

![[Pics/INE/15_36b1e50737.png]]

As you can see in the above image, `lookupDNS` is the only available operation:

![[Pics/INE/15_1_863a52ea01.png]]

Viewing the details for the `lookupDNS` operation:

![[Pics/INE/15_2_c04a7dfac1.png]]

The information on the web page reveals the input and output parameters expected by this operation. All this information could be inferred directly from the WSDL file as well.

Scroll down to view the sample request:

![[Pics/INE/15_3_8fbcbad25a.png]]

**Step 16:** Interact with the web service using Burp Suite.

Copy the sample request for the `lookupDNS` method to the Burp Repeater window:

![[Pics/INE/16_0c51cce4ac.png]]

Remove the `/mutillidae` part of the URL as the web service is located at `/webservice` and not at `/mutillidae/webservice`.

Once that is done, send the request:

![[Pics/INE/16_1_0a92371c82.png]]

We got back a 200 response!

Scroll down to view the DNS Lookup results:

![[Pics/INE/16_2_a4ecc73e8f.png]]

**Step 17:** Identifying command injection vulnerability.

Send a semicolon (`;`) instead of a hostname:

![[Pics/INE/17_92e46bbd0d.png]]

Scroll down to view the DNS Lookup results:

![[Pics/INE/17_1_d307a9ee03.png]]

Notice that the provided hostname was used as is and no errors were reported.

Send the following payload to the web service:

**Payload:**

```
;ls -al
```

Sending the above payload results in a 200 response:

![[Pics/INE/17_2_28d8e6840c.png]]

Scroll down to view the DNS Lookup results:

![[Pics/INE/17_3_3ae6977164.png]]

Notice that the output for the `ls -al` command is retrieved!

So the web service is indeed vulnerable to command injection vulnerability.

**Step 18:** Retrieve the flag.

Since the DNS Lookup web service is vulnerable to a command injection vulnerability, send the following command to find the flag file:

**Payload:**

```
;find / -iname *flag* 2>/dev/null
```

Sending the above payload results in a 200 response:

![[Pics/INE/18_4db1f01410.png]]

The output reveals that the third flag is present in the file `/app/flag3`.

Send the following payload to read the flag file:

**Payload:**

```
;cat /app/flag3
```

![[Pics/INE/18_1_a8869ab80d.png]]

**flag3:** 6ae5422a0f086335766e1de8f75c16b7

**So that was all about performing command injection attacks against web services.**

And with that, we conclude this lab on attacking web services. There was a lot of ground to cover in this lab and we learned quite a lot of techniques to pentest SOAP-based web services and bypass SOAP body restrictions.
