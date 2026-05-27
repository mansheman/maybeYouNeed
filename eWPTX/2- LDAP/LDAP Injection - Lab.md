2026-02-04 06:29

Status:

Tags:[[eWPTX]][[LDAP]]
###### Prerequisites: 
# LDAP Injection - Lab
today we are going to solve LDAP injection LAB from INE https://github.com/digininja/vuLnDAP

**Step 1:** Open the lab link to access the Kali GUI instance.

![[Pics/INE/1_a6e2d311eb.png]]

**Step 2:** Check if the provided machine/domain is reachable.

**Command:**

```
ping -c3 demo.ine.local
```

![[Pics/INE/2_241ccacae9.png]]

The provided machine is reachable.

**Step 3:** Check open ports on the provided machine.

**Command:**

```
nmap -p- demo.ine.local
```

![[Pics/INE/3_9654e95641.png]]

Ports 22 (SSH) and 9090 are open on the target machine. As mentioned in the challenge description, the vulnerable web application is available on port 9090.

**Step 4:** Check the web application available on port 9090.

Open the following URL in the browser:

**URL:** http://demo.ine.local:9090

![[Pics/INE/4_ec31e9cce0.png]]

A [vuLnDAP](https://github.com/digininja/vuLnDAP) instance is present on the target machine.

**Step 5:** Explore the web application.

Click on the **Stock Control** link:

![[Pics/INE/5_b7fb641609.png]]

Select the **Fruit** category:

![[Pics/INE/5_1_48e1fdc208.png]]

Notice the URL:

![[Pics/INE/5_2_c2c5fa99f7.png]]

The value **fruits** is reflected in the **objectClass** parameter.

# Exploiting LDAP injection vulnerability in the web application.

**Step 6:** Perform LDAP injection.

Set the value `*` in the **objectClass** parameter:

**Note:** `*` is a special character that would end up returning all the `objectClass` items.

![[Pics/INE/6_792a957cbd.png]]

![[Pics/INE/6_1_7f99fc73f2.png]]

Notice the output contains the names of the system users as well.

Click on **More Info** link for the administrator (user david):

![[Pics/INE/6_2_581ef67495.png]]

Click **Back**:

![[Pics/INE/6_3_199859aa8e.png]]

Notice the URL parameter **objectClass** contains the value **posixAccount**.

The page contains the list of system users present in the LDAP database.

**Step 7:** Extract SSH password for user david from the LDAP database.

Click on **More Info** link for the administrator (user david):

![[Pics/INE/7_237fdc7bbe.png]]

The resulting page doesn't contain any interesting information:

![[Pics/INE/7_1_f3accc2944.png]]

Check the LDAP schema information from the following URL:

**URL:** https://tldp.org/HOWTO/archived/LDAP-Implementation-HOWTO/schemas.html

![[Pics/INE/7_2_7b070b5984.png]]

Notice the **posixaccount** object class. Some of the attributes it contains are:

- uidNumber
- gidNumber
- homedirectory
- userpassword

Open the home page of the vulnerable web application:

**URL:** http://demo.ine.local:9090

![[Pics/INE/7_3_2194d31f43.png]]

The admins store the SSH keys in the database.

Search for the ssh keys in the posixaccount object class:

**Search Query:**  

```
ldap posixaccount ssh keys
```

![[Pics/INE/7_4_e968c72d29.png]]

Check the [StackOverflow](https://serverfault.com/questions/653792/ssh-key-authentication-using-ldap) link from the results:

![[Pics/INE/7_5_6f1abe5ae8.png]]

We can use the `sshPublicKey` attribute to get the SSH keys.

Use the following URL to get the uid, gid, home directory, user password, and SSH public key for the user david:

**URL:** http://demo.ine.local:9090/item?cn=david&disp=uidNumber,gidNumber,homedirectory,userpassword,sshPublicKey

![[Pics/INE/7_6_3ac1be1d0e.png]]

Notice the `sshPublicKey` returned a password instead of SSH keys. Probably this is the SSH password of the user.

**Potential SSH password for user david:** r0ck_s0l1d_p4ssw0rd

**Step 8:** Retrieve the flag from the target machine using the recovered SSH credentials.

Try connecting to the target machine over SSH using the following credentials:

**Username:** david  
**Password:** r0ck_s0l1d_p4ssw0rd

**Command:**

```
ssh david@demo.ine.local
```

![[Pics/INE/8_adc3da8e0d.png]]

Login was successful!

Retrieve the flag:

**Commands:**  

```
ls
cat flag
```

![[Pics/INE/8_1_5ce7691a25.png]]

**Flag:** 5520dd2d85e5003db92048c629bb5072

That was all for LDAP injections!