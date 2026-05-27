2025-08-27 01:26

Status:

Tags: [[eWPT]] [[File & Resource Attacks]]
###### Prerequisites: 
# Web Service Security Labs


**Step 1:** Open the lab link to access the Kali GUI instance.

![[Pics/INE/1_1ce97ec19d.png]]

**Step 2:** Check if the provided machine/domain is reachable.

**Command:**

```
ping -c3 demo.ine.local
```

![[Pics/INE/2_ca5d6b61f9.png]]

The provided machine is reachable.

**Step 3:** Check open ports on the provided machine.

**Command:**

```
nmap -sS -sV demo.ine.local
```

![[Pics/INE/3_6c71a77e24.png]]

Port 80 (Apache webserver) and 3306 (MySQL server) are open on the target machine.

**Step 4:** Open the browser to inspect the hosted website.

**URL:** http://demo.ine.local

![[Pics/INE/4_8bd2e93727.png]]

An instance of [**OWASP Mutillidae II**](https://github.com/webpwnized/mutillidae) is hosted on the Apache webserver.

**Step 5:** Open the **Lookup User** web service.

In the challenge description, the link for the web services is already provided.

You could navigate to the web services from the menus available on the web page:

Visit **Web Services** > **SOAP** > **Username Enumeration** > **Lookup User**:

![[Pics/INE/5_bfedbc98d6.png]]

Alternatively, you can visit the web service URL provided in the challenge description:

**URL:** http://demo.ine.local/webservices/soap/ws-user-account.php

And that should take you to the following web page:

![[Pics/INE/5_1_232b83e120.png]]

**Step 6:** Enumerate the WSDL file.

**Information:**  
WSDL stands for Web Services Description Language and is used to describe web services. It is written in XML.

A WSDL document describes a web service. It specifies the location of the service, and the methods of the service, using these major elements:

![[Pics/INE/6_3ac6dc4620.png]]

**Reference:** https://www.w3schools.com/xml/xml_wsdl.asp

And that's why this file is quite interesting since it's a description of the web service we will be pentesting.

We would be locating all the defined operations/methods of the service and the parameters they accept to invoke those later.

On the web service page, click on the **WSDL** file link or append `?wsdl` to the URL:

![[Pics/INE/6_1_074df822bd.png]]

That would show the WSDL file for the web service we will be interacting with:

![[Pics/INE/6_2_1c5b7370db.png]]

As we already saw on the [W3schools WSDL page](https://www.w3schools.com/xml/xml_wsdl.asp), `<portType>` describes the operations that can be performed by the web service along with the messages (or the parameters) that have to be passed.

If you check the WSDL for the web service, you will find that it supports five operations, namely:

- getUser
- createUser
- updateUser
- getAdminInfo
- deleteUser

![[Pics/INE/6_3_c78a3e0e14.png]]

![[Pics/INE/6_4_f56f972c18.png]]

The documentation for all of these operations is also available in the WSDL file.

The conclusion is that the web service page only lists 3 out of these 5 operations. So seemingly, these operations are not meant to be invoked by the normal users and are probably reserved for the administrators.

**Step 7:** Check information on `getUser` operation.

Head back to the web service page and click on the `getUser` link:

![[Pics/INE/7_791e278cac.png]]

As you can see, the information on the `getUser` operation is listed. The input, and output parameters are also listed here. All this information is also present in the WSDL file which we explored in the last step.

Scroll down to view the complete request to invoke the `getUser` method of the web service:

![[Pics/INE/7_1_399db4b8f6.png]]

**Step 8:** Launch Burp Suite.

Open the start menu and select: **03 - Web Application Analysis** -> **burpsuite**

![[Pics/INE/8_259809fec7.png]]

If you get a warning about the JDK version, feel free to ignore it and press the **OK** button:

![[Pics/INE/8_1_ecde6f6ec2.png]]

Create a temporary project:

![[Pics/INE/8_2_5493a91a93.png]]

We will be using the default Burp Suite configuration:

![[Pics/INE/8_3_9b607210bd.png]]

After these steps, Burp would start up!

![[Pics/INE/8_4_ec2d5f7ef4.png]]

**Step 8:** Invoke the `getUser` method.

Open the Repeater window in Burp Suite:

![[Pics/INE/9_fdb559d4df.png]]

Copy the request to invoke the `getUser` method from the web page and paste it in the Repeater window:

![[Pics/INE/9_1_40633912a0.png]]

Remove the `/mutillidae` part of the URL as the web service is located at `/webservice` and not at `/mutillidae/webservice`.

Once that is done, send the request.

That should open a dialog asking for the `Host` and the `Port` of the target machine:

**Host:** demo.ine.local **Port:** 80

![[Pics/INE/9_2_5ff00fbb8c.png]]

Now with the target configured, click on the send button again:

![[Pics/INE/9_3_473cd7cc10.png]]

Now the request worked!

Scroll down the response and you would notice the `username` and `signature` returned by the web service:

![[Pics/INE/9_4_2c4ae63ff1.png]]

**Step 10:** Invoke the `deleteUser` method.

Replace all the occurrences of `getUser` with `deleteUser` and send the request:

![[Pics/INE/10_cf622d02c8.png]]

As you can see in the above image, we got back a 200 response!

Scrolling down reveals about an error:

![[Pics/INE/10_1_6b66ca90e8.png]]

As you can see in the above image, there's a parameter that is missing from the request sent to the web service.

But more importantly, we were able to invoke the web service that was supposedly hidden.

Head over to the web service's WSDL file and locate `deleteUserRequest`:

![[Pics/INE/10_2_2e0e2266fc.png]]

As you can notice in the above image, there are two parameters accepted by this function, namely: `username` and `password`. Both of them are of the string data type.

**Step 11:** Performing SQL Injection attack.

Now we know that the `deleteUser` requires two parameters. But we don't know the password of any of the users.

We could perform a dictionary attack. Alternatively, we could check if the web service is vulnerable to SQL Injection.

Let's go with the latter. First, we will send a single quote (`'`) as the password and see if any errors are returned from the web service:

![[Pics/INE/11_16792e5412.png]]

As you can see, we again get back a 200 response.

Scrolling down reveals a SQL error:

![[Pics/INE/11_1_abceff4303.png]]

The SQL query executed by the web service is also revealed in the error message:

**SQL Query:**

```
SELECT username FROM accounts WHERE username='Jeremy' AND password=''';
```

As you can see, there is a single extra quote (`'`) which invalidated the whole SQL query.

So we are sure that an SQL Injection vulnerability is present in the web service!

Now we will send the following SQL injection payload:

```
' or '1'='1
```

The above payload would result in the following SQL query being executed by the web service:

**SQL Query:**

```
SELECT username FROM accounts WHERE username='Jeremy' AND password='' or '1'='1';
```

The condition on the right side of the `OR` clause would always evaluate to true since `'1'='1'` and therefore, even when the conditions on the left side evaluate to false, the result would still be true!

Therefore, sending the above payload in the password field should delete the account for the user named `Jeremy`:

![[Pics/INE/11_2_3e0d4988c8.png]]

Scrolling down reveals the account deletion message:

![[Pics/INE/11_3_f0e40af825.png]]

As you can see, we also get back the first flag:

**flag1:** 6701f2d8bf691da5ee694d3a1786a7e6

**Step 12:** Invoke the `getAdminInfo` method.

Check the WSDL file for the parameters required to invoke the `getAdminInfo` method:

![[Pics/INE/12_34c910169d.png]]

As you can see in the above image, no parameters are required to invoke the `getAdminInfo` method.

Scroll down to the view the documentation for the `getAdminInfo` method:

![[Pics/INE/12_1_680fa4e0b4.png]]

This method retrieves some (non-sensitive) account details for the admin user. A sample request has also been provided.

Head over to Burp Repeater and send the following request:

**Request:**

```
POST /webservices/soap/ws-user-account.php HTTP/1.1
Accept-Encoding: gzip,deflate
Content-Type: text/xml;charset=UTF-8
Content-Length: 387
Host: localhost
Connection: Keep-Alive
User-Agent: Apache-HttpClient/4.1.1 (java 1.5)
<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:ws-user-account">
<soapenv:Header/>
<soapenv:Body>
<urn:getAdminInfo soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
</urn:getAdminInfo>
</soapenv:Body>
</soapenv:Envelope>
```

![[Pics/INE/12_2_8ab6955d27.png]]

Sending the above request results in a 500 response!

Scrolling down reveals that the only admin user could invoke this method:

![[Pics/INE/12_3_a7a9bad9ff.png]]

So it seems like there is some restriction on the invocation of this method.

**Step 13:** Bypass the SOAP body restrictions using the SOAPAction header.

There is possibly a restriction on the invocation of the `getAdminInfo` method.

An alternative way to invoke a web service method is by using the SOAPAction header.

**Information:**  
The SOAPAction header is a transport protocol header (either HTTP or JMS). It is transmitted with SOAP messages, and provides information about the intention of the web service request, to the service. The WSDL interface for a web service defines the SOAPAction header value used for each operation. Some web service implementations use the SOAPAction header to determine behavior.

**Reference:** https://www.ibm.com/docs/en/baw/19.x?topic=binding-protocol-headers

Head over to the WSDL file for the web service and inspect the `soapaction` attribute for the `getAdminInfo` operation:

![[Pics/INE/13_36ef5150b4.png]]

**SOAPAction attribute value:**  

```
urn:ws-user-account#getAdminInfo
```

Now head over to the Burp Repeater window and send the following request:

**Request:**

```
POST /webservices/soap/ws-user-account.php HTTP/1.1
Accept-Encoding: gzip,deflate
Content-Type: text/xml;charset=UTF-8
Content-Length: 228
Host: localhost
Connection: Keep-Alive
User-Agent: Apache-HttpClient/4.1.1 (java 1.5)
SOAPAction: urn:ws-user-account#getAdminInfo
<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:ws-user-account">
</soapenv:Envelope>
```

As you can notice, we are sending the SOAPAction header and have removed the SOAP request header and body.

![[Pics/INE/13_1_fdef6b42cf.png]]

Sending the above request results in a 200 response!

Scrolling down the response shows the details for the admin user and the second flag:

![[Pics/INE/13_2_f0dd8e6229.png]]

**flag2:** 4315749f21a66fad53895daccbebf309

**So that was all about WSDL enumeration, invoking hidden methods, performing SQL injection and bypassing SOAP body restrictions.**

# Performing command injection attacks against the DNS Lookup web service.

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