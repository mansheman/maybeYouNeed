# Lab Environment

In this lab environment, you will have a GUI access to a Windows machine. The PCAP file for analysis is `mal_capture.pcapng`, located on the Administrator's desktop.

**Objective:** Analyze the captured traffic using Wireshark and complete the following tasks:

**Task 1: Detect Malicious TCP Traffic**

- Identify the IP that conducted port scanning
- Determine the range of ports that were scanned

---

**Task 2: Detect Malicious HTTP Traffic**

- Identify the vulnerable endpoint susceptible to SQL injection
- Determine the SQL injection payload used to dump data

---

**Task : Detect Malicious DNS Traffic**

- Identify zone transfer attempts
- Detect DNS tunneling

---

**Task 4: Detect Malicious ICMP traffic**

- Detect data exfiltration via ICMP
- Recover exfiltrated data

---

# Tools

The best tools for this lab are:

- Wireshark
**Step 1:** Access the Windows machine.

![[Pics/ecthp/wireshark-lab-multi-protocol/5daa833e993418a34937086fad61547fc8a7629777cc450617049dd85923e036-e01199fce7.png]]

**Step 2:** Start Wireshark and open the provided PCAP file.

![[Pics/ecthp/wireshark-lab-multi-protocol/2166b1c4e7d965ba0770a743b72d68f90c7586383d86e644c17603fc9485446d-220cee18b5.png]]

![[Pics/ecthp/wireshark-lab-multi-protocol/0f09d71def20d40938563c20b57172cf5b47615c3f2f0d05ac42fb8fa3c104b9-47fb53e0c8.png]]

![[Pics/ecthp/wireshark-lab-multi-protocol/6a84c76abd424a0abebccbb4b6c6d637559323fc3f09883a0f9d907b3a2fab1d-befecd129b.png]]

**Step 3:** Let’s spot some suspicious TCP traffic.

Port scanning over TCP occurs when attackers use tools like Nmap to probe a system’s ports in an attempt to discover open or vulnerable services. This type of scanning generates unusual TCP packets, often with specific flags like SYN or FIN, sent to multiple ports in rapid succession. Such behavior can indicate reconnaissance activity, as attackers attempt to map the network before launching further attacks.

To identify the IP address that conducted the port scanning, navigate to **Statistics > Conversations**.

![[Pics/ecthp/wireshark-lab-multi-protocol/2176488f3230ad27e946b5ff3721ee1cc301cace66e03746175ff9b96cbdff86-cb5e5483d9.png]]

And then, go to the **TCP** tab to view the conversation details.

![[Pics/ecthp/wireshark-lab-multi-protocol/7f8cecd814550f60c46cf1d252dac0129054d9b4bf84c3a0628cde066b0d05dd-bd8226f203.png]]

You will notice numerous requests from IP address **192.13.139.2**, all using the same source port (as shown in **Address A** and **Port A**), targeting different destination ports (listed under **Port B**) of a victim machine. This indicates that the attacker at **192.13.139.2** performed a port scan. To determine the range of scanned ports, you can sort the Port B column by clicking on the tab once.

![[Pics/ecthp/wireshark-lab-multi-protocol/a7637e1381e937459795cba0589671f24ce411c29e5748987e23781c843ed74a-ac4812bbb0.png]]

![[Pics/ecthp/wireshark-lab-multi-protocol/19b29ab2f23839df3d95ffa525b050e74bb47c303b70e14201f1637e158f4578-ad0783629c.png]]

The range of ports that were scanned is **1–9999**.

You can now close this window.

Another way to determine port scanning activity, specifically TCP SYN / stealth port scans is by using the following filter:

**Filter:**

```
tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size <= 1024
```

This filter looks for TCP packets with SYN flag set, ACK flag not set and the Window size <= 1024 bytes.

![[Pics/ecthp/wireshark-lab-multi-protocol/402f34ea6c833a0d9f7762e2cc964b769160dee243799229ca412fb5f3e5bd69-867c82bf2e.png]]

If an abundance of such packets is observed within a brief timeframe, it's probable that someone is engaging in SYN scans within our network, possibly by executing commands like `nmap -sS <target>`.

**Step 4:** Next, let’s take a look at the HTTP traffic. Apply the following filter:

**Filter:**

```
http
```

![[Pics/ecthp/wireshark-lab-multi-protocol/11a6a6d977574ef3a63e8027d4fb01fceac6895aa2a35e6232b5e6bd06e07be2-69ecbe6e75.png]]

We observe some legitimate HTTP traffic related to a web application. However, the source IP address of these requests is the same as the one that conducted the port scanning. Let’s investigate further to see if the attacker was attempting anything suspicious.

There is a **POST** request made to the login endpoint in frame **20126**. Right-click on it and select **Follow > HTTP Stream**.

![[Pics/ecthp/wireshark-lab-multi-protocol/412d0b780fe2f8ecf3ccad85bd7ebb51b0ba873c1d5a5982ffac108da2e2cfdb-2c5186701e.png]]

![[Pics/ecthp/wireshark-lab-multi-protocol/5cfbe95d7a4068803237dde72fa6213fc4b21e49ab79104353c429188ae37dde-8127040b34.png]]

Well, it looks like the attacker was able to log in using common credentials — **tester:tester**, as indicated by a redirect to `/dashboard`.

Going back to the HTTP traffic, if we scroll down further, we notice some unusual requests made to the `/emergency_physicians/1` endpoint. For example, take a look at frame **20522**. Right-click on it and select **Follow > HTTP Stream**.

![[Pics/ecthp/wireshark-lab-multi-protocol/8f07bb5a9d70f3e859b8280fb7826404c4be5efda340439ab8550fc2aa1bca5c-5970d7acba.png]]

![[Pics/ecthp/wireshark-lab-multi-protocol/db82c35966c91b6180490ee1dcf701e603f196090324323ea590481017f9e4be-6460e1274c.png]]

Check the `User-Agent` header. It indicates the use of **sqlmap**, a well-known automated SQL injection tool. Its presence alone strongly suggests malicious intent. The request appears to be an attempted SQL injection attack.

Since the cookie is also passed in the request, the `/emergency_physicians/1` endpoint appears to be an authenticated endpoint, one that the attacker accessed using the common credentials. So, this is an authenticated SQL injection (SQLi) attack attempt.

The server responded with a 500 Internal Server Error. If you scroll down to the bottom of the response, you’ll see the specific error: `sqlite3.OperationalError: ...`

![[Pics/ecthp/wireshark-lab-multi-protocol/e2eaad8924b4f44454fd2e3d9c2bff6e4bde3d04ab1e7400b53115d888492111-1def6083d7.png]]

This confirms that a SQL error occurred, meaning the request successfully reached the SQL layer and disrupted the query. That’s a clear indication of a vulnerability where the input wasn’t properly sanitized. Additionally, the error message reveals that the application’s backend database is **SQLite**.

Let’s see if the attacker succeeded with the attack. Scroll down further, right-click on frame **21247**, and select **Follow > HTTP Stream**.

![[Pics/ecthp/wireshark-lab-multi-protocol/2c10ca39cc9ee84021e183ec279e1d2d40ea599943fdeffec4032773bf0dc71f-d77b19d451.png]]

Copy the payload you see in the URL and decode it using any online URL decoder.

![[Pics/ecthp/wireshark-lab-multi-protocol/612bd7f77c0a42573230e35a4e2633a3c670160a2523f37cde0200c9ad8f0cae-04cf97346d.png]]

Injected payload:

```
-7324 UNION ALL SELECT NULL,'qxxpq'||JSON_GROUP_ARRAY(COALESCE(tbl_name,' '))||'qbqvq',NULL,NULL,NULL FROM sqlite_master WHERE type='table'-- bTNt
```

This payload is attempting to:

- Bypass normal application behavior via a `UNION SELECT`.
    
- Extract all **SQLite table names** from the internal `sqlite_master` table.
    
- Use `JSON_GROUP_ARRAY(COALESCE(tbl_name,' '))` to concatenate them.
    
- Wrap the result with custom markers: `qxxpq` and `qbqvq` to identify the injected result in the HTML response.
    

Scroll down to the bottom.

![[Pics/ecthp/wireshark-lab-multi-protocol/72bb11e15ae45721509425e382d37acde7b97af54201c7982375b9281c6cf38c-141db9bc3d.png]]

In the HTML output, as highlighted above:

```
<h2>Name: qxxpq[&#34;er_doc&#34;,&#34;users&#34;]qbqvq</h2>
```

The attacker successfully extracted and identified: `er_doc` and `users`. These are table names in the victim database, which were returned and rendered in the HTML response.

Similarly, check frame **21277**.

![[Pics/ecthp/wireshark-lab-multi-protocol/fad4387623e23201f16b294a36f34ba8a478185a56af45a3d037d36737eced8a-f204951eeb.png]]

Injected payload is:

```
-6927 UNION ALL SELECT NULL,MAX('qxxpq'||COALESCE(sql,' '))||'qbqvq',NULL,NULL,NULL FROM sqlite_master WHERE tbl_name='users'-- kNQx
```

What it did:

- It selected the schema definition of the `users` table from `sqlite_master`, which stores metadata about SQLite tables.
    
- `COALESCE(sql,' ')` returns the SQL CREATE TABLE statement.
    
- `qxxpq` and `qbqvq` are markers added to identify the injected result in the HTML response.
    

Checking the response:

![[Pics/ecthp/wireshark-lab-multi-protocol/444a2f48015addaf79099fe8b7705d65b0def882b24484631b8f77f80e97fe6f-1f40873b2a.png]]

In the HTML output:

```
<h2>Name: qxxpqCREATE TABLE users
    (id INTEGER PRIMARY KEY,
    username TEXT UNIQUE,
    password TEXT,
    isAdmin BOOLEAN,
    isActive BOOLEAN)qbqvq</h2>
```

The attacker successfully retrieved the SQL schema of the `users` table. The table contains the following columns:

- id (INTEGER PRIMARY KEY)
- username (TEXT, UNIQUE)
- password (TEXT)
- isAdmin (BOOLEAN)
- isActive (BOOLEAN)

Similary, check frame **21307**.

![[Pics/ecthp/wireshark-lab-multi-protocol/44ee21e675e657758a3aa4752260cf15afcbe89af59c7accc9c518dc7336ae33-fbefb70800.png]]

Injected payload is:

```
-2951 UNION ALL SELECT NULL,'qxxpq'||JSON_GROUP_ARRAY(COALESCE(isActive,' ')||'zyzmbe'||COALESCE(isAdmin,' ')||'zyzmbe'||COALESCE(password,' ')||'zyzmbe'||COALESCE(username,' '))||'qbqvq',NULL,NULL,NULL FROM users-- wXTL
```

What it does:

- Enumerates data from the `users` table.
- Uses `JSON_GROUP_ARRAY(...)` to concatenate the results of:
    - isActive
    - isAdmin
    - password
    - username
- Delimits using markers like `zyzmbe`, `qxxpq,` and `qbqvq` for easier parsing.

Checking the response:

![[Pics/ecthp/wireshark-lab-multi-protocol/9455370046c855d9ccbbdf6dd2cdd29c38eaab5ae97d2b77fd3127ec228ea898-4728fbc1e9.png]]

In the HTML output:

```
<h2>Name: qxxpq[&#34;1zyzmbe0zyzmbetesterzyzmbetester&#34;,&#34;1zyzmbe1zyzmbesh#ape@3392!zyzmbedcarlson&#34;,&#34;0zyzmbe0zyzmbetestingzyzmbetesting&#34;,&#34;0zyzmbe0zyzmbesam0yed123@1zyzmbesamboy&#34;,&#34;0zyzmbe1zyzmbec@y1areb!zyzmbecayla&#34;]qbqvq</h2>
```

The attacker successfully dumped the sensitive `users` table:

|isActive|isAdmin|password|username|
|---|---|---|---|
|1|0|tester|tester|
|1|1|sh#ape@3392!|dcarlson|
|0|0|testing|testing|
|0|0|sam0yed123@1|samboy|
|0|1|c@y1areb!|cayla|

We have successfully detected the malicious HTTP traffic.

**Step 5:** Moving on, let’s take a look at the DNS traffic. Apply the following filter:

**Filter:**

```
dns
```

![[Pics/ecthp/wireshark-lab-multi-protocol/efe1a9b368e86cbdc30bc268c1bbd54f03fde9761d6b302e604ddf6466f59bf1-8535c1c996.png]]

We observe some legitimate DNS lookups at the beginning. However, if you check frames **23738** and **23740**, you’ll notice a DNS zone transfer (AXFR) request.

- `AXFR`: A DNS query type used for zone transfers.
- `ine.local`: The domain name being requested — likely an internal network zone.

**What is AXFR?**

AXFR (Authoritative Zone Transfer) is a mechanism used to replicate DNS databases from a master server to a secondary (slave) server. It's usually used in internal DNS setups. But it’s sensitive! It transfers all DNS records for a domain, not just a single query like A or MX.

**Why is this a security concern?**

If a DNS server accepts unauthenticated AXFR requests from anyone, an attacker could:

- Dump all internal hostnames and IP addresses
- Map your internal network without port scanning
- Discover staging/dev/test environments

Let's check if the zone tranfer attempt was successful. Double-click on the response frame **23740**, expand the `Domain Name System (response)` section and then expand the `Answers` section.

![[Pics/ecthp/wireshark-lab-multi-protocol/8620fd5d3de3c7f59cdaed49cf29d255e71c9a3dbbf702c4548e5f8bdd353661-560074a579.png]]

The zone transfer attempt was indeed successful, as the response contains all the dumped DNS records, including subdomain information.

If you scroll down further, you’ll also observe a bunch of unusual DNS requests. The queries are prefixed with `dnscat`. This is a major red flag. It is very likely that DNS tunneling is in play.

![[Pics/ecthp/wireshark-lab-multi-protocol/8ee3d1487831d24cd7d5c257555acb4936f42071db001d904131c76df05ef753-8196d8bf47.png]]

**What is dnscat?**

**dnscat2** is a tool that creates a covert channel over DNS. It is used for:

- Command and control (C2)
- Data exfiltration
- Bypassing firewalls/proxies

It abuses the fact that DNS is often wide open on networks and not inspected closely.

The data is encrypted by default, so it might be difficult to uncover exactly what was stolen. However, the presence of **dnscat** queries strongly suggests that data exfiltration took place over DNS, which is a common tactic used to evade detection in traditional network monitoring systems.

**Step 6:** Next, let’s take a look at the ICMP traffic. Apply the following filter:

**Filter:**

```
icmp
```

![[Pics/ecthp/wireshark-lab-multi-protocol/26cfe5158f0597370360a91814e031c833f3c5f28151c17ae79f876efeab81b2-7d0d9f0168.png]]

We observe some legitimate PING requests at the beginning. However, starting from frame **29380** to **29464**, we can see something suspicious unfolding.

We notice a series of ICMP Echo requests and replies between two machines. In the frame **29380**, the **Data** section reveals the string `BOFstoleninfo.txt`, which likely indicates malicious activity taking place, possibly data exfiltration disguised within ICMP traffic.

![[Pics/ecthp/wireshark-lab-multi-protocol/8663ebb0517b09a05d3a45a4b4f12e25e4cf77de07d31ed56a76f1afffd095aa-aba16b4011.png]]

In frame **29382**, the data string `SEND` likely represents a command issued by the server, instructing the compromised victim to begin transmitting data.

![[Pics/ecthp/wireshark-lab-multi-protocol/ffb1f81a4f62e9894957be85fdc69f34354889ea8f244c61a7c23a0464c89c9d-94e7132fb4.png]]

Frame **29457** contains what appears to be exfiltrated content in plain text, likely part or all of a file named `BOFstoleninfo.txt`.

![[Pics/ecthp/wireshark-lab-multi-protocol/96bb2657cb2f39e6e8b8bd6d3def61e1bfd386dee455c7a56e8d93b74e258808-268fa46bbe.png]]

In frame **29459**, the string `OK` serves as an acknowledgment from the server, confirming it received the previously transmitted data successfully.

![[Pics/ecthp/wireshark-lab-multi-protocol/ad60ffee82f8f45ff3c49567e6e13139b450a9bf334cfee4a5603ba90034339e-acc8787ac1.png]]

Next, frame **29462** contains the string `EOFstoleninfo`, which likely signals the end of the file transfer, indicating that the victim has completed sending the file.

![[Pics/ecthp/wireshark-lab-multi-protocol/f3a37f2106b5e68770053d9e692c62bc917d662f7f6201cdedba6d4213b421b7-8cb5e1632b.png]]

Finally, frame **29464** contains the word `COMPLETE`, which seems to be a final confirmation from the server, acknowledging that the entire data transfer process was successfully completed.

![[Pics/ecthp/wireshark-lab-multi-protocol/695256ecf86dc04708ee5af3c79892e6d0ab8e59c8028bb0d265b014cf8ffb1f-94da6da6a7.png]]

This pattern shows a command-response cycle, where data is embedded within the payload of ICMP Echo requests and replies, indicating data exfiltration over ICMP.

# Conclusion

In this lab, we analyzed a captured network traffic file using Wireshark to uncover signs of malicious activity across multiple protocols. We identified a host performing extensive TCP port scanning, targeting a wide range of ports in an attempt to discover vulnerable services. Further inspection of HTTP traffic revealed that the attacker exploited a web application using common credentials and later launched a successful SQL injection attack, exposing sensitive data. The DNS traffic exposed a successful zone transfer, leaking internal subdomain information, along with suspicious DNS tunneling activity likely used to exfiltrate data. Finally, we observed ICMP echo traffic containing embedded filenames such as BOFstoleninfo.txt, indicating data exfiltration through covert ICMP channels. These findings demonstrate how attackers can use different layers of the network stack to carry out reconnaissance, exploit vulnerabilities, and exfiltrate data, all while remaining stealthy.

# References

- [https://www.wireshark.org/]
- [https://github.com/iagox86/dnscat2]
- [https://sqlmap.org/]
- [https://meyerweb.com/eric/tools/dencoder/]
- 