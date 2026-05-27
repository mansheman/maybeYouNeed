
# Lab Environment

In this lab environment, you will have GUI access to a Kali machine. The Splunk instance should be accessible at `http://splunk.advanced.local:8000`. Use the `ecthp` index for all queries.

**Objective:** Perform threat hunting in Splunk to complete the following tasks:

1. On the SECLOGS machine, the attack originating from the Kali system involved targeting both local and domain accounts. Can you separate these two types and display them in a table containing the following fields: targeted host, targeted user, targeted domain and source workstation name?
    
2. Can you also calculate the number of failed login attempts for each local and domain account, and display the count in a separate column?
    
3. On the SECLOGS machine, the attacker created a malicious service on June 23, 2025, around 10:41 AM. Identify the service and determine its intended function.
    
4. Find all websites that were browsed using Internet Explorer (IE) on the SECLOGS machine, and sort them by time and user.
    

**Logs to Check:** Security, System and Sysmon

# Tools

The best tools for this lab are:

- Splunk
- Firefox
**Step 1:** Access the Kali machine.

![[Pics/ecthp/advanced-splunk-lab/fc805972b41128d4edc1b531e28fe034803f632eb1d1bf62cd5b6ff79491190c-bb942475cc.png|1400]]

**Step 2:** Navigate to the following URL to access Splunk:

**URL:**

```
http://splunk.advanced.local:8000
```

![[Pics/ecthp/advanced-splunk-lab/97db5d0909e8e788211775922e9102656279d405a3ee3768b5f76007bda397eb-4aec3cee25.png|1400]]

Go to **Search and Reporting**.

![[Pics/ecthp/advanced-splunk-lab/aeabaef553264967f232d6c6dac4f0eddef4249ef87ec5d4f24cb38fdca26a8f-0c07205790.png|1400]]

**Step 3:** Let's begin our hunt by examining the failed logon events on the SECLOGS machine in the Windows Security logs. Make sure to set the time frame to **"All time"**.

**Query:**

```
index=ecthp source="WinEventLog:Security" EventCode=4625 host=SECLOGS
```

![[Pics/ecthp/advanced-splunk-lab/5707118a0c6122647a9d2fa19a45e70129eb239b0da03cb332e53f18d34521a6-c762b265f3.png|1400]]

Expand to see the event details.

**Key Details:**

- **Event Type:** Logon Failure
    
- **Attempted Account:** `sysoperator`
    
- **Failure Reason:** Unknown user name or bad password
    
- **Source Host:** kali (likely the attacker's machine)
    
- **Source Network Address:** - (missing, possibly due to logging limitations)
    

The targeted account is `sysoperator` which appears to be a **domain account**, as the **Account Domain** field is `research` which matches the Active Directory domain name of the instance (**seclogs.research.SECURITY.local**).

Local accounts would display the local machine name (e.g., SECLOGS or CLIENT) as the domain, or the domain field could be empty instead.

However, if you were to apply a filter for targeted local accounts only or targeted domain accounts only, you would need to construct a query that examines both the `Account Domain` and `Account Name` fields under **Messsage** in each event. Note that in Event ID 4625, these fields appear twice — once under **'Subject:'** and once under **'Account For Which Logon Failed:'**.

Our focus is on the second occurrence of these fields, specifically under **'Account For Which Logon Failed:'**. Because this section shows the actual account the attacker or user attempted to log in as — i.e., the targeted account. The first occurrence (under `Subject:`) shows the source account initiating the attempt.

Begin by extracting all occurrences of the `Account Name` and `Account Domain` fields from the events for more clarity.

**Query:**

```
index=ecthp EventCode=4625 host="SECLOGS"
| rex field=Message max_match=0 "Account Name:\s+(?<all_users>\S+)"
| rex field=Message max_match=0 "Account Domain:\s+(?<all_domains>\S+)"
| table all_users, all_domains
```

**Explanation:**

`index=ecthp EventCode=4625 host="SECLOGS"`:

- Searches your data in the ecthp index.
    
- Filters for Windows Security Event ID 4625, which corresponds to a failed logon attempt.
    
- Limits results to the host named SECLOGS.
    

`rex field=Message max_match=0 "Account Name:\s+(?<all_users>\S+)"`:

- `rex` is a command used to apply a **regular expression (regex)** to extract data from a field — here, from the **Message** field.
    
- `max_match=0` tells Splunk to find all matches, not just the first.
    
- The pattern is:
    

```
Account Name:\s+(?<all_users>\S+)
```

- This matches any string starting with **"Account Name**:"
    
- Followed by one or more spaces (\s+)
    
- Followed by a non-space string (\S+) → this is captured in the new field called **all_users**.
    
- Because `max_match=0`, all occurrences of **Account Name** in the message are captured into a multi-value field called **all_users**.
    

`rex field=Message max_match=0 "Account Domain:\s+(?<all_domains>\S+)"`:

- Does the same thing, but for **Account Domain**.
    
- Captures all domain values into the multi-value field **all_domains**.
    

`| table all_users, all_domains`:

- Displays the result in a table showing all captured usernames and domains for each event.
    
- Both **all_users** and **all_domains** may show multiple values (e.g., Account Name in Subject and in Account For Which Logon Failed).
    

![[Pics/ecthp/advanced-splunk-lab/2f6a4d76f1f5c40c9413bede961fa6647a20f21c5b2f44c2ee7e4d8ba73fd662-1e29c2627f.png|1400]]

We notice that the first occurrence of both the `Account Name` and `Account Domain` fields shows a hyphen (**“-”**). The second occurrence of each reveals the correct information. However, in few cases, we observe that while the `Account Name` field is present in the second occurrence, the corresponding `Account Domain` field is empty — causing the next string, **“Failure”**, to appear in its place.

![[Pics/ecthp/advanced-splunk-lab/28717f8452e09938dc112129056da4cc9c7010f506db7484cbd49f21a5a2d18a-e38c982b91.png|1400]]

The string **"Failure"** is only displayed in the events where domain is empty and the local account is targeted.

**Step 4:** We have ruled out the first occurrence of these fields. Now, let's display only the second occurrence of the `Account Name` and `Account Domain` fields.

**Query:**

```
index=ecthp EventCode=4625 host="SECLOGS"
| rex field=Message max_match=0 "Account Name:\s+(?<all_users>\S+)"
| rex field=Message max_match=0 "Account Domain:\s+(?<all_domains>\S+)"
| eval target_user=mvindex(all_users, 1)
| eval target_domain_raw=mvindex(all_domains, 1)
| table target_user, target_domain_raw
```

**Explantion (additional fields):**

`eval target_user=mvindex(all_users, 1)`:

- Extracts the second value (index 1) from the **all_users** array.

`eval target_domain_raw=mvindex(all_domains, 1)`:

- Same logic — picks the second value from the **all_domains** array.

`table target_user, target_domain_raw`:

- Creates a table showing just the targeted user and their domain.

![[Pics/ecthp/advanced-splunk-lab/e1d85e3c4ce1b54fcb90a3939d3e832b9faee7d4b10fdfb60f7629ea6e7af89d-57a2ced5ab.png|1400]]

Next, let's prepare a query that displays only the local accounts targeted.

**Query:**

```
index=ecthp EventCode=4625 host="SECLOGS"
| rex field=Message max_match=0 "Account Name:\s+(?<all_users>\S+)"
| rex field=Message max_match=0 "Account Domain:\s+(?<all_domains>\S+)"
| eval target_user=mvindex(all_users, 1)
| eval target_domain_raw=mvindex(all_domains, 1)
| eval target_domain=if(like(target_domain_raw, "Failure"), "", target_domain_raw)
| eval is_local=if(target_domain="" OR target_domain == "SECLOGS", "local", "domain")
| where is_local="local"
| table host, target_user, target_domain, Workstation_Name
```

**Explanation (additional fields):**

`| eval target_domain=if(like(target_domain_raw, "Failure"), "", target_domain_raw)`

- Handles noisy/garbled logs where the domain might have the word "Failure" due to unusual formatting.
    
- If **target_domain_raw** contains **"Failure"**, it sets **target_domain** to an empty string.
    

`| eval is_local=if(target_domain="" OR target_domain == "SECLOGS", "local", "domain")`

- Defines whether the targeted account is **local or domain**:
    
    - If target_domain is empty or equal to the host name (SECLOGS), it is classified as local.
    - Otherwise, it’s a domain account.

`| where is_local="local"`

- Filters the results to only show failed logon attempts against local accounts.

`| table host, target_user, target_domain, Workstation_Name`

- Displays a table with:
    
    - The host (SECLOGS)
        
    - Targeted username
        
    - Target domain
        
    - Workstation name (the system initiating the attempt)
        

![[Pics/ecthp/advanced-splunk-lab/e7da7c3a33983816c0ec867f828f87b648ed677ec4761eb192691f303400029d-3eaf9e83d6.png|1400]]

Similarly, to display only domain accounts targeted in failed login attempts, you just need to modify this line:

```
| where is_local="local"
```

to

```
| where is_local!="local"
```

In Splunk, `!=` is a comparison operator that means **"not equal to."**

This line filters the results to include only events where the `is_local` field is not equal to "local" — i.e., only domain accounts in your case.

**Query:**

```
index=ecthp EventCode=4625 host="SECLOGS"
| rex field=Message max_match=0 "Account Name:\s+(?<all_users>\S+)"
| rex field=Message max_match=0 "Account Domain:\s+(?<all_domains>\S+)"
| eval target_user=mvindex(all_users, 1)
| eval target_domain_raw=mvindex(all_domains, 1)
| eval target_domain=if(like(target_domain_raw, "Failure"), "", target_domain_raw)
| eval is_local=if(target_domain="" OR target_domain == "SECLOGS", "local", "domain")
| where is_local!="local"
| table host, target_user, target_domain, Workstation_Name
```

![[Pics/ecthp/advanced-splunk-lab/b9ec04abc8f4381080081731803861a0dc27719b9040a276bf5dd678aa68d92f-ec399d7f87.png|1400]]

**Step 5:** Now, let's count the number of failed attempts on each local account and display the results in a separate column:

**Query:**

```
index=ecthp EventCode=4625 host="SECLOGS"
| rex field=Message max_match=0 "Account Name:\s+(?<all_users>\S+)"
| rex field=Message max_match=0 "Account Domain:\s+(?<all_domains>\S+)"
| eval target_user=mvindex(all_users, 1)
| eval target_domain_raw=mvindex(all_domains, 1)
| eval target_domain=if(like(target_domain_raw, "Failure"), "", target_domain_raw)
| eval is_local=if(target_domain="" OR target_domain == "SECLOGS", "local", "domain")
| where is_local="local"
| stats count by host, target_user, target_domain, Workstation_Name
```

**Explanation (additional fields):**

`| stats count by host, target_user, target_domain, Workstation_Name`

- Groups the results by:
    
    - host
        
    - target_user (username that failed)
        
    - target_domain
        
    - Workstation_Name (originating machine)
        

![[Pics/ecthp/advanced-splunk-lab/64ce5fb46485f537abe1a637fbcd18176bdfbf2ce75600caae8baeb88468da8e-aea7ba8cb2.png|1400]]

So, there have been a total of **18** failed login attempts targeting the local user account `secadmin`.

Similarly to count the number of failed attempts on each domain account and display the results in a separate column:

**Query:**

```
index=ecthp EventCode=4625 host="SECLOGS"
| rex field=Message max_match=0 "Account Name:\s+(?<all_users>\S+)"
| rex field=Message max_match=0 "Account Domain:\s+(?<all_domains>\S+)"
| eval target_user=mvindex(all_users, 1)
| eval target_domain_raw=mvindex(all_domains, 1)
| eval target_domain=if(like(target_domain_raw, "Failure"), "", target_domain_raw)
| eval is_local=if(target_domain="" OR target_domain == "SECLOGS", "local", "domain")
| where is_local!="local"
| stats count by host, target_user, target_domain, Workstation_Name
```

![[Pics/ecthp/advanced-splunk-lab/c0b3bc29c04ef186ee9fc67ccf3a71d086e4e3baa04014ef3baca7e287493ce8-5a004ee7ff.png|1400]]

So, there have been a total of **17** failed login attempts targeting the domain account `sysoperator`.

**Step 6:** Next, we’ll investigate and identify the malicious service created by the attacker on the SECLOGS machine.

**Query:**

```
index=ecthp LogName="Microsoft-Windows-Sysmon/Operational" EventCode=12 host=SECLOGS EventType=CreateKey TargetObject="HKLM\\System\\CurrentControlSet\\Services\\*"
| table _time, host, User, Image, TargetObject, EventType
| sort -_time
```

This query is designed to identify registry key creation events related to Windows services — a common technique used for persistence by attackers.

**Explanation:**

`index=ecthp LogName="Microsoft-Windows-Sysmon/Operational"`

- Searches within the Splunk index named **ecthp**. Filters for logs specifically from the **Sysmon Operational log**.

`EventCode=12`

- Filters for Event ID 12, which in Sysmon means:
    
    - Registry key creation (i.e., CreateKey).

`host=SECLOGS`

- Limits the search to the SECLOGS host (the monitored machine)

`EventType=CreateKey`

- Adds another filter to confirm that the type of action was creating a registry key.

`TargetObject="HKLM\\System\\CurrentControlSet\\Services\\*"`

- Narrows down to registry keys created under the Windows Services path, which is:

```
HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\
```

- This is a critical location: Windows uses it to define system services and their behavior (such as what binary to execute at boot).

`| table _time, host, User, Image, TargetObject, EventType`

- Selects and displays only the following fields in a clean table:
    
    - `_time`: When the event occurred
        
    - `host`: The machine name (SECLOGS)
        
    - `User`: Which user created the key
        
    - `Image`: The executable that performed the action
        
    - `TargetObject`: The exact registry key that was created
        
    - `EventType`: Should always be **CreateKey** in this case
        

`| sort -_time`

- Sorts the final output in reverse chronological order (newest events first).

![[Pics/ecthp/advanced-splunk-lab/b1cc5f024db7170ccb6f7db48f755ecce01551ae7cc8d868e88394206219e970-f8f43f6b0e.png|1400]]

At 10:41:59 AM on June 23, 2025, a new registry key was created under:

```
HKLM\System\CurrentControlSet\Services\FunnyService
```

This is a key part of how Windows services are registered. The event indicates that a suspicious service named `FunnyService` was likely being installed or set up.

Next, check Event ID 13 in Sysmon.

**Query:**

```
index=ecthp LogName="Microsoft-Windows-Sysmon/Operational" EventCode=13 host="SECLOGS" TargetObject="HKLM\\System\\CurrentControlSet\\Services\\FunnyService\\*"
| table TargetObject, Details
```

**Explanation:**

`EventCode=13`

- This is Sysmon Event ID 13, which means:
    
    - A registry value was set.
- It's triggered when a value (not a key) in the registry is modified or created — for example, setting the **ImagePath** for a Windows service.
    

`TargetObject="HKLM\\System\\CurrentControlSet\\Services\\FunnyService\\*"`

- Filters to show only registry value modifications inside the **FunnyService** registry key (which defines a custom or malicious Windows service).
    
- The `*` wildcard matches any subkey or value name under that path (like ImagePath, Start, etc.).
    

`| table TargetObject, Details`

- This formats the output as a table with:
    
    - `TargetObject`: The full registry path and value name that was modified.
        
    - `Details`: The new value that was set, such as the executable path for the service.
        

![[Pics/ecthp/advanced-splunk-lab/ab41d6c138e4327a98edfc7eb31d8b08c57fd527da9739bb0db9fbdf9a5fe3b4-17d0e59ff7.png|1400]]

**Registry Key Path and Meaning:**

`HKLM\System\CurrentControlSet\Services\FunnyService\ObjectName`

- **Details:** LocalSystem
    
- This means the service runs under the LocalSystem account, which has very high privileges.
    

`...FunnyService\DisplayName`

**Details:** Windows Updates

- This is a deceptive display name. It pretends to be a legitimate system service.

`...FunnyService\ImagePath`

- **Details:** C:\Users\Public\reverse.exe
    
- This is the path to the binary that will run when the service starts. In this case, it's a suspicious executable, likely a reverse shell or malware.
    

`...FunnyService\ErrorControl`

- **Details:** DWORD (0x00000001)
    
- Indicates how the system responds if the service fails to start:
    
    - **0x1** means the system logs the error but continues with startup

`...FunnyService\Start`

- **Details:** DWORD (0x00000002)
    
- Indicates automatic startup during system boot.
    
    - 0x2 = Auto Start.

`...FunnyService\Type`

- **Details:** DWORD (0x00000010)
    
- This denotes the service type:
    
    - 0x10 = User-mode service that runs in its own process (not shared with other services).

**This indicates that an attacker or malware created a persistent Windows service named FunnyService:**

- Disguised as a system update (Windows Updates),
    
- Runs a suspicious binary (reverse.exe) from a public folder,
    
- Starts automatically at boot with high privileges.
    
- This is a classic persistence technique used by attackers.
    

The same can be observed in the Windows System logs:

**Query:**

```
index=ecthp EventCode=7045 source="WinEventLog:System" host="SECLOGS" 
| table _time, host, ComputerName, Message
```

![[Pics/ecthp/advanced-splunk-lab/2538ae47d0e5c9427abed6a19ae841b6f6223442b8c4ae3d81b36e9d0e103b08-b51d272807.png|1400]]

**Step 7:** To find out all websites visited via Internet Explorer (IE) on the SECLOGS machine by a user, we can use the following query:

**Query:**

```
index=ecthp LogName="Microsoft-Windows-Sysmon/Operational" EventCode=22 Image="C:\\Program Files\\Internet Explorer\\iexplore.exe"
| table _time, host, User, Image, QueryName, QueryStatus
| sort -_time User
```

**Explanation:**

`EventCode=22`: Sysmon Event ID 22 is for DNS query events.

`Image="C:\Program Files\Internet Explorer\iexplore.exe"`: Only DNS queries made by Internet Explorer are shown.

`| table _time, host, User, Image, QueryName, QueryStatus`

- This selects and displays:
    
    - `_time`: When the DNS query occurred.
        
    - `host`: The system where the query happened.
        
    - `User`: The user account under which the process ran.
        
    - `Image`: The executable making the DNS query.
        
    - `QueryName`: The actual domain name being queried (e.g., malicious-site.com).
        
    - `QueryStatus`: Whether the DNS query was successful or not (e.g., 0 = Success).
        

`| sort -_time User`

- `_time`: Sorts by _time in descending order (most recent first).
    
- `User`: Sorts alphabetically within each timestamp if multiple users have events at the same time.
    

This Splunk query analyses DNS query activity using Sysmon Event ID 22, specifically made by Internet Explorer (iexplore.exe).

Since, most website visits start with a DNS query to resolve the domain name, this query gives a list of domains the browser tried to access.

![[Pics/ecthp/advanced-splunk-lab/d3e7a465e14e08456b7827faba462f7fea651cd843850c6ffe00d6697a6a95c3-5ceafe4a36.png|1400]]

We see numerous events. Notable ones include visits to the following websites made by the `secadmin` user. Assuming that the compromised user on this machine had logged on to these web services and opted to stay logged in, browser-stored cookies may still be valid. In that case, attacker can do the following:

- **amazon.com**
    
    - Hijack the session to access order history, stored cards, or pivot to AWS services for infrastructure abuse.
- **netflix.com**
    
    - Access watch history to profile user interests.
- **dropbox.com**
    
    - Access, upload, or sync malicious files through the user's cloud storage for persistence or data theft.
- **drive.google.com**
    
    - Steal or plant sensitive documents, search for passwords, and abuse shared folders to pivot laterally.

Additionally,

- **wetransfer.com**
    
    - Silently exfiltrate sensitive files using the trusted user’s session without raising suspicion.
- **pastebin.com**
    
    - Post stolen data or C2 commands to evade attribution and monitoring.

# Conclusion

In this lab, we focused on building complex and sophisticated Splunk queries to analyze Windows Security, System, and Sysmon logs. The objective was to understand how to extract meaningful insights from raw log data by writing precise, multi-step queries. This approach enabled us to trace attacker behavior, detect anomalies, and uncover persistence mechanisms. This lab highlights how powerful and flexible Splunk can be when used to craft tailored searches for advanced threat detection and investigation.

# References

## - https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/
