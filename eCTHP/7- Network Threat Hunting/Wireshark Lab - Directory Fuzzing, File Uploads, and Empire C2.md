# Lab Environment

In this lab environment, you will have a GUI access to a Windows machine. The PCAP file for analysis is `sus_capture.pcapng`, located on the Administrator's desktop.

**Objective:** Analyze the captured traffic using Wireshark and complete the following tasks:

**Task 1: Detect Directory Fuzzing and Malicious File Uploads**

- Identify the IP address performing directory fuzzing to determine valid endpoints.
- Identify the endpoint discovered by the attacker that allows insecure file uploads.
- Identify the malicious file uploaded through the endpoint and determine its payload type.

---

**Task 2: Detect C2 Traffic**

- Detect C2 channels based on various static and behavioral traits: default URIs, user agents, server responses, and beaconing behavior.

---

# Tools

The best tools for this lab are:

- Wireshark
  
  
  **Step 1:** Access the Windows machine.

![[Pics/ecthp/wireshark-lab/412f58df784cbc71f25a83e2d8b309e3e585f0fb636cf234c4136dbda79c0b5a-bb16798119.png]]

**Step 2:** Start Wireshark and open the provided PCAP file.

![[Pics/ecthp/wireshark-lab/8663f67f8c14f3740823d76cbc5a4de4b89fcc25bb6e533a3f381079c811b432-8cba1b248a.png]]

![[Pics/ecthp/wireshark-lab/93c15c3273b5507db31c7da79623a60903a378f2df71aab069098977148a46ed-8100c0d1ac.png]]

![[Pics/ecthp/wireshark-lab/09d0e5ae4272399421d69f5b8356ee234bc4a8858f239989a20ac51648ccd0ce-78a5e0954b.png]]

**Step 3:** We see a lot of traffic related to different protocols. Let's filter and view only HTTP traffic using the following Wireshark filter:

**Filter:**

```
http
```

![[Pics/ecthp/wireshark-lab/9f9618f20a01a97d0d44892b337a0fb5c1b0e46e9976668e6bcbc64656d9a204-e0f77589cb.png]]

We see a lot of GET requests being made to random endpoints from **10.10.31.2**. Right-click on the first frame, then navigate to **Follow > TCP Stream**.

![[Pics/ecthp/wireshark-lab/0fd5b71852cc7a7569f8c73762d362ad907666400eb595c87c55a02fe8177799-0e8247ae93.png]]

![[Pics/ecthp/wireshark-lab/800be6d3ee0bf13c21959b55330bc2f0bc579e2fdb7b4d9fdd167f18ab1ed1e2-a0a1ea1211.png]]

We can observe the entire conversation—the requests made and the corresponding responses received.

Note the **User-Agent** header in the GET requests. It is a custom User-Agent header used by **FFuF (Fuzz Faster U Fool)**, a popular web fuzzing tool. The FFuF version appears to be specifically compiled or packaged for Kali Linux.

Also, note the **Server** header in the response.

**What is Web Fuzzing?**

Web Fuzzing is an automated technique that sends random data to an application to detect misconfigurations, unexpected behavior, and hidden parameters. FFuF is one such tool used for web fuzzing. It is commonly used for:

- Brute-forcing directories and files
    
- Testing for hidden endpoints
    
- Fuzzing parameters
    

**Step 4:** Use the following filter to view the server's **200 OK** responses to the attacker's fuzzing:

**Filter:**

```
http.response.code == 200 && http.server contains "Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.2.12"
```

Wireshark filters support logical operators to combine multiple conditions. These operators are used to refine and narrow down the packets we’re interested in based on more complex criteria.

In the above filter,

- `http.response.code == 200`: This condition matches HTTP response packets with a status code of 200, indicating that the request was successful.
    
- `&&` (Logical AND): The `&&` operator is used to combine two conditions. Both conditions must be true for a packet to be selected. In this case, the response code must be 200 and the server must match the specified signature.
    
- `http.server contains "Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.2.12"`: This condition filters packets where the `http.server` field contains the specific string "Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.2.12", identifying the server software and version.
    

In summary, we are selecting packets that indicate a successful endpoint discovery by the attacker during fuzzing.

![[Pics/ecthp/wireshark-lab/7218b33408d18db94a0df2b440dcad38e6d23f2a96d6ae789b1d17b6873ca97e-267fa1da65.png]]

Select the first frame. In the bottom-left pane, expand **Hypertext Transfer Protocol**. Right-click on `[Request URI: /dashboard/]` and select **Apply as Column**.

![[Pics/ecthp/wireshark-lab/539239215dbf01bd71a7b69f966109016672f601d7b9bc47d70354ab8de9393b-402452d341.png]]

![[Pics/ecthp/wireshark-lab/5cdd0b3c361ec9900b67eb00eef15447f47a5e1d2a207527048e35646e7ce2b5-5bc88afb7d.png]]

This displays all the endpoints that the attacker successfully discovered. They are:

- /dashboard/
    
- /img/
    
- /icons/
    
- /xampp/
    
- /uploads/
    
- /upload.php
    

**Step 5:** The attacker discovered two interesting endpoints: `/uploads/` and `/upload.php`. One is a directory that likely contains uploaded files, and the other is a PHP webpage that likely allows file uploads. Let's use a filter to view requests related to these endpoints.

**Filter:**

```
http.request.uri contains "/uploads/" || http.request.uri contains "/upload.php"
```

Here,

- `http.request.uri contains "/uploads/"`: This condition matches HTTP request URIs that contain the string "/uploads/".
    
- `||` (Logical OR): The `||` operator is used to combine two conditions, where **either** of the conditions must be true for the packet to match. In this case, it will match packets where the URI contains either "/uploads/" or "/upload.php".
    
- `http.request.uri contains "/upload.php"`: This condition matches HTTP request URIs that contain the string "/upload.php".
    

This filter captures HTTP requests where the URI contains either "/uploads/" or "/upload.php", commonly targeting file upload endpoints. The `||` operator ensures that either condition is sufficient for the packet to match.

![[Pics/ecthp/wireshark-lab/e2e1758aacd9156376339b608a6fc0b938f2cb22486c5ea900b3eb3075693794-ac92b46535.png]]

We notice a few GET requests to `/uploads/` and `/upload.php`, as well as one POST request to `/upload.php`.

Right-click on frame **86711** containing the **POST** request, then navigate to **Follow > TCP stream**.

![[Pics/ecthp/wireshark-lab/0dc804e2fe4d22fa74a2803407380842cbf36c182c82166797c4f90ef561d3e9-4bfe708dc9.png]]

![[Pics/ecthp/wireshark-lab/36aea8002d57140027b2f7e87d2563c66ef58df3677cb8e9852a19a76cdf8d6b-7a9db2b9b4.png]]

The attacker uploaded a file named **update.bat**, which looks suspicious. It appears to contain a PowerShell payload. Here's what we can determine from its contents:

- **Batch Script Execution**
    
    - The script starts with `2>NUL & @CLS & PUSHD "%~dp0"`, which suppresses error messages, clears the screen, and sets the working directory to the script's location.
        
    - It then executes `powershell.exe` with `-nol -nop -ep bypass`, which disables the PowerShell logo, prevents profile loading, and bypasses PowerShell's execution policy to allow unrestricted script execution.
        
    - The script deletes itself after execution with `DEL "%~f0"`.
        
- **Encoded PowerShell Payload**
    
    - The script includes a `-enc` (encoded) PowerShell command, meaning the actual payload is Base64-encoded and obfuscated.
- **Potential Malicious Behavior**
    
    - The payload may likely includes PowerShell commands for system reconnaissance, persistence, or even downloading and executing remote code.

The request was successfully accepted, as seen in the 200 OK response below. This likely indicates that the application lacked the necessary protections to prevent malicious file uploads.

![[Pics/ecthp/wireshark-lab/85c04517b44a8df7fc0c057c583f0f26f0eb983825313ba79a9bbce38046830a-34a4ba8972.png]]

Navigate to the next stream. You will observe that the **update.bat** file was successfully dropped in the `/uploads` directory. The attacker likely confirmed this because directory listing is enabled for `/uploads`.

![[Pics/ecthp/wireshark-lab/17756911e49c9a3fec5c2803cfc6c6855c6b63b5e4011767c5e96ff05e6ed675-d84fcfba40.png]]

Please note that the infected machine's IP address is **10.0.21.105**.

**Step 6:** There is a high chance that the malicious file was executed. Let's investigate what might have happened next. First, filter out the FFuF-related requests.

**Filter:**

```
http.request.method == "GET" && !(frame contains "User-Agent: Fuzz Faster U Fool v1.3.1 Kali Exclusive <3")
```

Here,

- `http.request.method == "GET"`: Matches HTTP requests that use the GET method.
    
- `&&` (Logical AND): Ensures both conditions must be true — the request must be a GET request and the next condition must also be satisfied.
    
- `!` (Logical NOT): Negates the condition that follows it.
    
- `frame contains "User-Agent: Fuzz Faster U Fool v1.3.1 Kali Exclusive <3"`: Checks if the packet contains this specific User-Agent string, which is typically used by the FFuF fuzzing tool. The `!` in front means we are **excluding** such packets.
    
- `!(...)`: Parentheses are used to group the negated condition.
    

This filter selects only GET requests that do not come from the FFuF fuzzing tool by filtering out packets that contain its User-Agent string. This helps us focus on other relevant traffic.

![[Pics/ecthp/wireshark-lab/4900b67a542855bcd89a992cf4093c6b89d545492761084e2e0a52fb305edd2b-b27bc21782.png]]

We observe multiple GET requests with unusual URIs originating from the infected machine to the attacker's machine. These URIs are:

- /admin/get.php
    
- /news.php
    
- /login/process.php
    

**PowerShell Empire**, a well know C2 framework is known to use these URIs to disguise C2 channels.

**Empire Source Code Reference:** [https://github.com/EmpireProject/Empire/blob/e37fb2eef8ff8f5a0a689f1589f424906fe13055/data/agent/agent.ps1#L78]

![[Pics/ecthp/wireshark-lab/6549e18e9cc3fbbedccc8c6a5538699d57f75bcbcaf5471329c639f2ce30697e-48d48dcdca.png]]

**What is C2 (Command and Control)?**

Command and Control Infrastructure, also known as C2 or C&C, refers to a set of tools and techniques used by attackers to remotely control compromised devices in a cyberattack. It allows them to issue commands, exfiltrate data, and maintain persistence inside a victim's network. Adversaries commonly attempt to mimic normal, expected traffic to avoid detection. There are many ways an adversary can establish command and control with various levels of stealth depending on the victim’s network structure and defenses. Some popular C2 frameworks include PowerShell Empire, Cobalt Strike, Sliver C2, Havoc and Merlin.

**Read More:** [https://attack.mitre.org/tactics/TA0011/]

**Step 7:** Let's further analyze the traffic to confirm that PowerShell Empire is in use. First, narrow down the traffic to these suspicious URIs:

**Filter:**

```
http.request.method == "GET" && ip.src == 10.0.21.105 && (http.request.uri contains "/news.php" || http.request.uri contains "/admin/get.php" || http.request.uri contains "/login/process.php")
```

Here,

- `http.request.method == "GET"`: Matches HTTP requests using the GET method.
    
- `ip.src == 10.0.21.105`: Filters packets originating from the IP address 10.0.21.105.
    
- `&&`: Logical AND — all conditions connected by && must be true.
    
- `||`: Logical OR — at least one of the conditions must be true.
    
- **Grouped URIs:**
    
    - `http.request.uri contains "/news.php"`
        
    - `http.request.uri contains "/admin/get.php"`
        
    - `http.request.uri contains "/login/process.php"`
        
    - These are grouped with `()` and connected using `||` to match if the request targets **any one** of these paths.
        

This filter captures GET requests from IP 10.0.21.105 that are directed to any one of the three specific URIs. Logical operators `&&` and `||` help combine multiple conditions, allowing us to precisely target potentially interesting or suspicious requests from a known source.

![[Pics/ecthp/wireshark-lab/58a4919feec8a6fd192c2341eaa53d2f3cfb4d255b5823516bd6678336395d99-92ae3b035e.png]]

Right-click on the first frame and navigate to **Follow > TCP Stream**.

![[Pics/ecthp/wireshark-lab/6dc1f78a24b1925aecba1d9d13e06900614ad2caaf8803c3e504d326a7ff933b-c5ce7a2bf0.png]]

Note the default User-Agent used by Empire:

- `User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko`

**Empire Source Code Reference:** [https://github.com/EmpireProject/Empire/blob/e37fb2eef8ff8f5a0a689f1589f424906fe13055/data/agent/agent.ps1#L78]

![[Pics/ecthp/wireshark-lab/302a014a07e37a0ca8e0f4f639c7fb86de18c68c22746141d26af3b5cfb49bf3-de70a48eee.png]]

Empire operates in two phases:

1. Staging Phase
    
2. Command / Post-Exploitation Module Execution Phase
    

**1. Staging Phase:** During the staging phase, the stager (PowerShell payload that we saw earlier) is executed on the victim's machine, attempting to establish a connection with the Empire Command and Control (C2) server. At this stage, the victim and the server exchange cryptographic keys, which will later be used to encrypt and decrypt all future communications.

When analyzing this phase in Wireshark, we can observe a sequence of three request-response exchanges between the victim and the C2 server:

- **Stage 0:** The victim first sends an HTTP GET request with a unique cookie in the header. The server responds with a 200 OK, delivering a large encrypted payload.

![[Pics/ecthp/wireshark-lab/39b63be55714b46e4172df83270035cf2994b13f483e587ca96dcd8a5cfb06c5-2d796b45d2.png]]

- **Stage 1:** The victim follows up with an HTTP POST request, containing an encrypted message. The server replies with another 200 OK, this time with a smaller response payload.

Scroll down further.

![[Pics/ecthp/wireshark-lab/b6f151579766eff5d094ed51fbc598f948c2e3fedc2f829ad50bbedb0798ab2c-0a4042095e.png]]

- **Stage 2:** The victim then sends a final POST request, prompting the server to respond with an even larger encrypted payload, completing the staging process.

Scroll down further.

![[Pics/ecthp/wireshark-lab/6baae47b9803fc56994ae649ffda9dc5c1e93397230e963babac48437877716f-c4cb8af5ad.png]]

At the end of this exchange, the victim is successfully registered with the Empire C2, and the system is ready for command execution and post-exploitation activities.

**2. Command / Post-Exploitation Module Execution Phase:** After the staging phase, the victim now periodically sends GET requests (beacons) to check for tasks. There are two possible scenarios:

- If the server has no tasks for the victim, the victim continues sending GET beacons. In response, the server returns a 200 OK default response.

Scroll down further.

![[Pics/ecthp/wireshark-lab/9d47b18fa73a6f0742501d2f62a26f0118b0b6e46b50a520c9f5a4cbd73aa6e4-69b6ae6e20.png]]

You should see multiple GET beacons with 200 OK default responses, indicating that the agent is idle and periodically checking for tasks.

Note that the Empire server uses a default header: `Microsoft-IIS/7.5`

**Empire Source Code Reference:** [https://github.com/EmpireProject/Empire/blob/e37fb2eef8ff8f5a0a689f1589f424906fe13055/lib/listeners/http_com.py#L116]

![[Pics/ecthp/wireshark-lab/ecaee627e57f9040668b424517e2af42296baddb14aa8a853e06d87851c13b55-10ca6f50a2.png]]

The default response includes a 404 page.

![[Pics/ecthp/wireshark-lab/52df25763390c4e8e78bb33f41ee7dba31d7208e0f6e0cf685b31934bc18719b-63040d27f2.png]]

**Empire Source Code Reference:** [https://github.com/EmpireProject/Empire/blob/e37fb2eef8ff8f5a0a689f1589f424906fe13055/lib/listeners/http_com.py#L144]

![[Pics/ecthp/wireshark-lab/91c80a2472aa3614f190de11293c5e34df6d7ae6b3d36b75ae9e0f8a716c37a9-5a6c2b4549.png]]

This beaconing can also be visualized.

First, apply the following filter:

**Filter:**

```
http.request.method == "GET" && ip.src == 10.0.21.105 && (http.request.uri contains "/news.php" || http.request.uri contains "/admin/get.php" || http.request.uri contains "/login/process.php")
```

And then navigate to **Statistics > I/O Graphs**.

![[Pics/ecthp/wireshark-lab/2615cf0298b9520f16411132df1a43aa670e2a4b0ff8ae9bdf9c6fbf594e1f98-386ed210e6.png]]

Uncheck any other filters and ensure that only the specified filter is applied.

**NOTE:** If you don't see the graph below, simply click the **Reset** button at the bottom left and uncheck/check your filter again.

![[Pics/ecthp/wireshark-lab/30453849b6d1ffc6d7e63b989a924a94506831b7e747be83a64d2aeef761c563-dea86ecb33.png]]

What we observe are GET beacons occurring approximately every 5 seconds, which is the default beaconing (callback) interval for an Empire agent.

- In the second scenario, if the Empire C2 server wants to issue tasks, such as running commands or executing post-exploitation modules:
    
    - The victim periodically sends HTTP GET beacons to the server, checking if there are any pending tasks. If the server has a command or module for the victim to execute, it responds with a payload containing the instructions.
        
    - In some cases, the victim may send a POST request to inform the server that the task has started. This step is optional and depends on the specific module being executed. The server acknowledges this with a 200 OK default response.
        
    - Once the task is completed, the victim sends the output of the executed command/module back to the server via a POST request. The server then replies with another 200 OK default response, confirming receipt.
        

If you scroll down further, you will encounter such a scenario, as shown below:

![[Pics/ecthp/wireshark-lab/cd371362e81b18367502b5378ec72582d2b379abe1101967bd3ebca3e9abd8e8-802565e7d1.png]]

Finally, we can draw the following conclusions from our analysis:

- The attacker performed directory fuzzing on a web application to discover valid endpoints. They identified a vulnerable upload endpoint that allowed malicious file uploads.
    
- The attacker most likely uploaded a PowerShell Empire stager, which was successfully executed on the victim machine.
    
- The attacker utilized PowerShell Empire C2 channels to execute commands/modules and may have exfiltrated sensitive data.
    

This analysis demonstrates how Wireshark can be used to investigate attacker activity and identify potential security breaches.

# Conclusion

In this lab, we analyzed network traffic with Wireshark to identify directory fuzzing attempts, detect malicious file uploads, and uncover potential command-and-control (C2) activity. We examined attacker behavior to determine the IP address performing fuzzing, the insecure upload endpoint discovered, and the nature of the uploaded payload. Additionally, we analyzed C2 communication patterns by inspecting default URIs, user agents, server responses, and beaconing behavior, allowing us to recognize and understand how attackers establish persistence and execute remote commands.

# References

- [https://www.wireshark.org/]