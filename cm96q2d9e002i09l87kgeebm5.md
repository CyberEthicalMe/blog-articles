---
title: "HTB Sherlock: Litter"
seoTitle: "HTB Sherlock: Litter Write-up"
seoDescription: "PCAP analysis and data extraction over alternative protocol - T1048. Cats do litter."
datePublished: Mon Apr 07 2025 07:00:28 GMT+0000 (Coordinated Universal Time)
cuid: cm96q2d9e002i09l87kgeebm5
slug: htb-sherlock-litter
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1711122601672/4bfd8038-f493-463e-91b6-3106378f4cff.png
tags: dns, soc, ethical-hacking, pcap, dfir

---

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">This write-up is a part of the <a target="_blank" rel="noopener noreferrer nofollow" href="https://blog.cyberethical.me/series/htb-sherlocks" style="pointer-events: none">HTB Sherlocks series</a>. Sherlocks are investigative challenges that test defensive security skills. I encourage you to try them out if you like digital forensics, incident response, post-breach analysis and malware analysis. <a target="_self" rel="noopener noreferrer nofollow" href="https://blog.cyberethical.me/go-htbapp" style="pointer-events: none">Are you ready to start the investigation?</a></div>
</div>

# Incident Details

**Name**: [Litter](https://blog.cyberethical.me/go-htbapp)  
**Category**: SOC  
**Difficulty**: Easy ([*Solved*](https://labs.hackthebox.com/achievement/sherlock/555018/555))

> Khalid has just logged onto a host that he and his team use as a testing host for many different purposes, it’s off their corporate network but has access to lots of resources in network. The host is used as a dumping ground for a lot of people at the company but it’s very useful, so no one has raised any issues. Little does Khalid know; the machine has been compromised and company information that should not have been on there has now been stolen – it’s up to you to figure out what has happened and what data has been taken.

%%[support-cta] 

# Evidences

> All evidence files are marked as readonly right after acquiring and their hash is written down. Read-only attribute does not affect the hash of a file.

01: ZIP archive, password protected (`hacktheblue`)

```plaintext
$ litter.zip
89ffb7db178388aad03ff1434279e28426be2421d8df6fc9c3afbee94de23b0d
```

02: PCAP file

```plaintext
$ suspicious_traffic.pcap
830c2c76e8055c6d807f642c684f716e3be73ddc33403d83c85f486086da47ca
```

# Analysis

I've glanced through the PCAP file while having in mind that we have data exfiltration scenario. The first two things that come to my mind are DNS exfiltration and simply uploading data via TCP.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1711051138645/0d981250-908e-41b9-a1f9-99f0dd34bbdd.png align="center")

These shouldn't look like that!

1. At a glance, what protocol appears to be suspect in this attack?
    

DNS!

2. There seems to be a lot of traffic between our host and another, what is the IP address of the suspect host?
    

Wireshark &gt; Statistics &gt; Endpoints. Select IPv4, sort from the highest Packets.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1711051155256/6c593f75-0676-43ef-b3af-7b7590438c91.png align="center")

3. What is the first command the attacker sends to the client?
    

Technically, it is possible to read what was transmitted using Wireshark display filter

```plaintext
ip.src == 192.168.157.144 and ip.dst == 192.168.157.145 and dns
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1711051167418/e17e9745-484d-4d4c-ba74-ecaa395f1218.png align="center")

But there is a [Python tool](https://github.com/josemlwdf/DNScat-Decoder) that helps to extract that information from PCAP file.

```plaintext
$ python /opt/dnscat/dnscat_decoder.py suspicious_traffic.pcap "microsofto365.com" > dnscat_dump.txt
```

All following questions can be answered purely from the dump.

4. What is the version of the DNS tunneling tool the attacker is using?
    
5. The attackers attempt to rename the tool they accidentally left on the client’s host. What do they name it to?
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1711122010891/89f6acc9-ed6f-4876-b233-33e8a9c15a4c.png align="center")
    
6. The attacker attempts to enumerate the user’s cloud storage. How many files do they locate in their cloud storage directory?
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1711122025835/3c686067-0b87-4a1e-b03f-0e8e76f97c1a.png align="center")
    
7. What is the full location of the PII file that was stolen?
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1711122044252/5082e8a6-9fc8-44df-b989-f2c577b9f109.png align="center")
    
8. Exactly how many customer PII records were stolen?
    

Dump shows 720 records, but starts from 0 s, so the answer is 721.

# Data Recovery

None required.

# Lessons Learned

Because this challenge was a bit short on analysis part, let's stay a bit on DNS exfiltration.

## DNS Data Exfiltration

**MITRE ATT.CK Technique**: [T1048](https://attack.mitre.org/techniques/T1048/)

Data exfiltration is all about flying under the radar. You can steal data simply by copying it to some removable drive (which is not [that uncommon](https://eu.usatoday.com/story/money/2022/04/06/cash-app-data-breach/9490327002/)). You can upload to cloud service or network share - but these leaves very verbose fingerprint on TCP/UPD communication, network logs etc. The challenge is here that usual protocols can and are easily set up to be monitored against data leakage. Attackers started to get creative and thought - maybe we try to use something that generates a lot of traffic so will not raise an alert, and something that is commonly and widely used, so can't be blocked and won't look suspicious.

DNS protocol meets those standards (that come to my mind, definitely there are more of them). And there were times when no one monitored DNS traffic - because what for? It is more like a technical thing, "how the Internet works" thing - domain names act as mnemonics for specific IP addresses, so something has to resolve them. We can't drop it, we can't stop DNS resolution at all. And each system, each software, can generate a lot of DNS queries on daily routine. Windows Update, loading CSS or JS on pages, virus signatures updates, requesting data from services - all of these happen via named mnemonics, not IP addresses.

So how attackers figured out the way to use it as a form of connection between [C&C](https://www.trendmicro.com/vinfo/us/security/definition/command-and-control-server) and a victim? This is to be honest quite simple and easy to explain.

1. Assume that malware is already present on the victim's server.
    
2. Encode data (C&C command response, file, anything) to hex-encoded blocks, as if requesting DNS server to resolve the domain - conforming one of two formats (*really* oversimplified)  
    `<encoded data>.<domain>` or `<tag>.<encoded data>`
    
3. Pass the prepared message as a DNS request to the attacker's server.
    

That's why in this challenge, we could eyeball the suspiciously looking DNS request - no-one is asking to resolve the following domain name: `7768656e6d6f7265636861696e7361776d616e736572696573.microsofto.com`

# Additional readings

%%[follow-cta] 

* [NIST Computer Security Incident Handling Guide](https://www.nist.gov/privacy-framework/nist-sp-800-61)
    
* [GitHub: dnscat2 protocol](https://github.com/iagox86/dnscat2/blob/master/doc/protocol.md)