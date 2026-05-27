# Lab Environment

In this lab environment, you will have GUI access to a Windows machine preconfigured with the necessary tools. A PCAP file named `maltraffic01.pcapng` is available on the Desktop for your analysis.

A system belonging to **Fincorp Technologies**, identified by the IP address **172.31.28.165** is suspected of being compromised. Your task is to use Wireshark to analyze the captured network traffic and identify signs of malicious activity.

**Objective:** Analyze the PCAP file to uncover malicious activity in the captured network traffic.

# Tools

The best tools for this lab are:

- Wireshark
- CyberChef
**Step 1:** Access the Windows machine.

![[Pics/ecthp/wireshark-lab-http-c2/4abf0d69e78ab5e0c52ef55db20936530706e26aecfc3159f5bebb84c58379df-9e10442070.png]]

**Step 2:** Start Wireshark and open the provided PCAP file.

![[Pics/ecthp/wireshark-lab-http-c2/401f3747fc9a04a9a62458d3ddf858c5e0267e531488cb234477edba399583a6-a9df1219a5.png]]

![[Pics/ecthp/wireshark-lab-http-c2/5108bfa899e3ad97f1d912dfdc28b2492eee5ef98bdb0a624c00b7ec4551aed9-7616a051a8.png]]

![[Pics/ecthp/wireshark-lab-http-c2/507a20bb38c2b13cb194c584d0ea31331b3d4d7fdea4a7ac7ded74fe4054d4f4-d2dee080f4.png]]

**Step 3:** Let's begin our investigation by examining any HTTP traffic.

**Filter:**

```
http
```

![[Pics/ecthp/wireshark-lab-http-c2/089ed99e2e5e316ab1a0280caebff93871bad9af8232dc5573950f587e661ad9-03ef02affd.png]]

Note that there has been repeated communication between the compromised machine and the IP **13.229.225.164**.

Let's check if the machine resolved **13.229.225.164** through a domain, to determine whether it corresponds to a known domain.

**Filter:**

```
dns
```

![[Pics/ecthp/wireshark-lab-http-c2/9f19d3c246b1761b5164bd9d3884b460fbbad438cd15971cc7fa03ae3f4581fe-0d20f1ebd4.png]]

Yes, the domain mapped to the IP address is `updatesrv.strangled.net`. However, we are not yet certain whether it is malicious.

Let’s return to analyzing the HTTP traffic.

**Step 4:** Right-click on the first frame, and select **Follow > TCP Stream**.

![[Pics/ecthp/wireshark-lab-http-c2/29075b6d4dfc19bd863115747f0038f3c01934d699ca8543ea9084289cf3705a-87973c9ddc.png]]

![[Pics/ecthp/wireshark-lab-http-c2/520d0d2033eee4961079b6d425e02179ed4597e4f03f2c3a2e4d07723429c968-26c023c092.png]]

What we see is a **GET** request made from the compromised machine to an unknown endpoint: `/b1x1521-1962-4b50-2c7z-94253dyq1341?cid=officedesk01`. The `cid` parameter most likely represents the hostname of the machine or client.

We observe an interesting response that says: **AccessCode Sent**, which likely marks the initiation of the conversation.

Similarly analyze the subsequent frames.

![[Pics/ecthp/wireshark-lab-http-c2/3e92e7ad2af10fd7b90568002b7dcb789c9de9a82422ad95cedd7b36f755b8e2-fa53d401dd.png]]

We observe multiple GET requests to the endpoint `/job?cid=officedesk01`, with the server responding with empty 200 responses. This strongly indicates beaconing behavior, which most likely suggests that Command and Control (C2) communication is taking place.

**Request Observations:**

- **GET /job?cid=officedesk01**
    
    - Client is polling the C2 for a “job” or command, identifying itself as `officedesk01` — likely a hostname.
- **User-Agent: python-requests**
    
    - Strong indicator this request came from a script (Python malware or implant), not a browser.
- **Host: updatesrv.strangled.net:8000**
    
    - Custom port (8000), suggests non-standard service, likely a C2 or custom backdoor listener.

**Response Observations:**

- **Server: BaseHTTP/0.6 Python/3.12.3**
    
    - Suggests the server is a Python-based HTTP server
- **200 OK with no payload**
    
    - Likely means: no job assigned at this time (beaconing with no instruction).

**In conclusion, this traffic indicates the following:**

- A Python-based implant is reaching out to a likely C2 server.
    
- It’s identifying itself (cid=officedesk01) and polling for tasks (/job).
    
- The C2 server is idle or hasn’t assigned any job (empty response body).
    

**Step 5:** However, as we continue scrolling down, we observe a deviation in the pattern.

Right-click on frame number **11992**, and select **Follow > TCP Stream**.

![[Pics/ecthp/wireshark-lab-http-c2/48bc20350632f78cabdb7d8810edfba21d752093b8d52bf819579b3a580e3ed3-68897cb771.png]]

A notable difference is observed in the response, which now contains a base64-encoded string (`IyAqPj4sY3ozKTM=`), unlike the previous empty responses.

**What This Likely Means?**

- The C2 server has issued a command to the compromised client (officedesk01).
    
- The implant is now expected to:
    
    - Decode this string.
    - Interpret/execute the resulting command.

Check the next stream.

![[Pics/ecthp/wireshark-lab-http-c2/1a2cb7e31c00595bc3f8fbde6983611856622f5af4dc0a8aee86edf2001c4c2a-01a1ac890c.png]]

![[Pics/ecthp/wireshark-lab-http-c2/0dd3152734fbced899a096154f413c32c82abd268b06c8958f4e81c10f0ef3cd-58681a1c28.png]]

This `POST /output` request is most likely the compromised client sending data back to the C2 server — typically a result of executing a command issued earlier via `GET /job`.

The body starts with `OUTPUT:`, followed by a long **base64-like blob**. This is likely the encoded output of whatever command the implant executed.

Let's use **CyberChef** to decode these. Go to the following link on your local machine: **https://gchq.github.io/CyberChef/**

Drag and Drop the `From Base64` recipe. Paste the string that we saw earlier in the input: `IyAqPj4sY3ozKTM=`

![[Pics/ecthp/wireshark-lab-http-c2/54ffa1319207c40dd4ae1221e070c4236f23f1340f6948845ea33478f42523c4-c69679ea60.png]]

The decoded output remains unclear, suggesting that the payload may be encrypted or deliberately obfuscated using a key. Identifying this missing key is essential to fully decode the content.

**Step 6:** While checking the DNS queries, we noticed something suspicious.

**Filter:**

```
dns
```

![[Pics/ecthp/wireshark-lab-http-c2/937e261655b8695819158b0e0a533f7fee443ef1569ed47e353c612a891bc84a-85fb716649.png]]

A DNS query and response exchange is observed between the compromised system and the suspected attacker IP address **13.229.225.164**.

The following A type DNS query is being generated from the compromised system:

```
VEhFX1NFQ1VSRV9YT1JfS0VZX0ZPUl9DMl9FTkNSWVBUSU9OX1RSQU5TTUlTU0l.PTl9PVkVSX0NIQU5ORUxfX19fX18.strangled.net
```

An 'A' (Address) record query is the most basic type of DNS query, used to request the IPv4 address corresponding to a domain name. However, in our case, the query appears long and suspicious — the domain being queried is clearly encoded and is likely part of data exfiltration or command-and-control activity.

Let's try to base64-decode the string before the second dot:

```
VEhFX1NFQ1VSRV9YT1JfS0VZX0ZPUl9DMl9FTkNSWVBUSU9OX1RSQU5TTUlTU0l.PTl9PVkVSX0NIQU5ORUxfX19fX18
```

![[Pics/ecthp/wireshark-lab-http-c2/dd1ef0961ab83c579067d5987c10ecd691d2f601bd138055e8fe96097b84de5b-6be59e5bcd.png]]

It appears that we have successfully identified the **key** used for Command-and-Control (C2) communication. The key is:

```
THE_SECURE_XOR_KEY_FOR_C2_ENCRYPTION_TRANSMISSION_OVER_CHANNEL__
```

This indicates that the key is an **XOR key**, which implies that **XOR-based encryption** is being used.

**XOR-based encryption** is a simple yet fundamental symmetric encryption technique that uses the bitwise XOR (exclusive OR) operation to both encode and decode data. It works by combining the plaintext with a secret key using the XOR operation, producing encrypted data that can only be decrypted with the same key. In our case, the encrypted output is further encoded using Base64.

**Step 7:** Now in addition to the `From Base64` recipe, search for and add the `XOR` recipe in CyberChef.

Paste the key and set the format to **UTF8**.

![[Pics/ecthp/wireshark-lab-http-c2/fbbb3556fe2a65e060c2c7ead43966ef2b4b7e3177cf3595ad8b386729cabbb8-f6bf7998ca.png]]

Now, paste the encoded command observed in the earlier GET request into the input field.

**Encoded/Obfuscated Command:**

```
IyAqPj4sY3ozKTM=
```

![[Pics/ecthp/wireshark-lab-http-c2/5e4a9f46acafa96d97adfda742da09aac010cd0c44887f51d82c407a61176c11-4b65d998f3.png]]

**Decoded command:**

```
whoami /all
```

Success! We have successfully decoded the command. This command provides detailed information about the current user, their security identifiers (SIDs), and privileges.

Similarly, try to decode the output sent in the earlier POST request.

![[Pics/ecthp/wireshark-lab-http-c2/f67b650544a68cf94e4c7736dffb232bce7f8924c1bf7a94a3ace079bf7685c6-98287118e1.png]]

Success!

Following the same approach, go through all the remaining requests and responses to uncover both the commands executed by the attacker and the results sent.

You will observe the following:

**Command and Result:**

![[Pics/ecthp/wireshark-lab-http-c2/4877a64a77e5ead973036ae5682016d83c7c453c8fcbb6015980a37fe91b8caf-626662e4e9.png]]

![[Pics/ecthp/wireshark-lab-http-c2/3ce97dc2bc309251f210e1a5dbf425cb3ded2f050d3097e718e2262349c3a579-cdc7950553.png]]

This command displays the list of users who are members of the local "Administrators" group on the system.

**Command and Result:**

![[Pics/ecthp/wireshark-lab-http-c2/fba9b788d9bba4f46502c1760a68e605c90203282b1e3bce9cebaa6254d0c965-99fe58f10a.png]]

![[Pics/ecthp/wireshark-lab-http-c2/b1965d5aa2eaee34d1741b3b0d1d65ba339204b80b5400eac472bf23f109c56f-ca01f30a4b.png]]

This command disables (turns off) Windows Defender Firewall for all network profiles (Domain, Private, and Public).

**Command and Result:**

![[Pics/ecthp/wireshark-lab-http-c2/abab1b62f8a2dca21349a88c28c890025c4f2a1693bff7b2b872cf1bbfba1e04-7d8be41dda.png]]

![[Pics/ecthp/wireshark-lab-http-c2/1e7019be6e7f3dcc81531265534b20cdc9b59a2d3b5d34a6f188b0fe65acad0e-2795e6c992.png]]

This command displays a list of saved credentials stored by the current user on the system.

**Command and Result:**

![[Pics/ecthp/wireshark-lab-http-c2/58b4023a259eb2f092c05cc88d63c3eb63c04e75aaf2b902fa59030e0f68ba87-1f18d2bd6e.png]]

![[Pics/ecthp/wireshark-lab-http-c2/63f55e425e0f1f7a334bfe5acd5e39d2bdb3f4039515f2bae13fba74b7b5fb30-c78c505cc4.png]]

This command creates a new local user account named "ducky" with the password "pass".

**Command and Result:**

![[Pics/ecthp/wireshark-lab-http-c2/14b73773950c7c4d9339676ca4fcb414a5e84f9ab3b401e483d2563c91be8250-7e5ebb5463.png]]

![[Pics/ecthp/wireshark-lab-http-c2/d2dd595dbbd74484d385d93f5a1982607ace9649492b295a63b0d256165d9c30-de603c87e0.png]]

This command adds the user "ducky" to the local "Administrators" group, giving it admin privileges.

The attacker seems to have disabled the firewall and established a backoor with full system access via user "ducky"!!

**Step 8:** The presence of a custom Command-and-Control (C2) communication channel has been confirmed. However, one critical aspect remains unclear: how did the attacker initially gain a foothold on the system?

Upon analyzing multiple protocols, we identified noteworthy activity associated with the **IMAP** protocol. IMAP (Internet Message Access Protocol) is a standard email protocol used by email clients to access and manage emails stored on a mail server.

**Filter:**

```
imap
```

![[Pics/ecthp/wireshark-lab-http-c2/ad1e42ad73ea0d8a97d3d2b2072e2f5b186d14019534600939c053e8bbae5508-3a40d6cb71.png]]

We see the compromised machine interacting with a mail server at **54.169.28.158**.

Right-click on the first frame, and select **Follow > TCP Stream**.

Scroll down a little.

![[Pics/ecthp/wireshark-lab-http-c2/ad00cbe1fb37cf71593e5465dd45965e086c74fd81053eeb52f099cb10fb9f5e-8e3145b562.png]]

You will notice that an employee with the email ID `jane.smith@staff.fincorp.com` received an urgent-sounding email from a suspicious address, `badguy@finc0rp.com`, instructing her to download and run the attached executable file named `update.exe` (likely malicious). Notice the subtle domain spoofing in the attacker's email address (**finc0rp.com** instead of **fincorp.com**), designed to trick the user’s eye.

This observation points clearly to a classic phishing attack as the possible initial foothold vector used by the attacker.

# Conclusion

In this lab, we successfully identified and analyzed a custom Command-and-Control (C2) communication channel established between a compromised system and an external attacker-controlled server. Through careful inspection of HTTP and DNS traffic, we uncovered signs of beaconing behavior and Base64-encoded payloads, which were later determined to be encrypted using XOR-based encryption.

Further investigation revealed that the initial compromise occurred via a phishing email containing a malicious attachment (update.exe), sent from a spoofed domain. Once executed, this file likely established initiated communication with the attacker's server.

By decoding both GET and POST request data, we were able to reconstruct the sequence of commands issued by the attacker and the system’s responses. This exercise highlights the importance of monitoring network traffic, understanding protocol behavior, and recognizing signs of common attack vectors such as phishing and data obfuscation.

# References

## - [https://gchq.github.io/CyberChef/]