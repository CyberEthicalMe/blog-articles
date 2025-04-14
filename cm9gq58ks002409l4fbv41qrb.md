---
title: "HTB Sherlock: Noted"
seoTitle: "HTB Sherlock: Noted Write-up"
seoDescription: "Data extorsion analysis. Notepad++ timestamps parsing. Full write-up for Hack The Box Sherlock box."
datePublished: Mon Apr 14 2025 07:00:24 GMT+0000 (Coordinated Universal Time)
cuid: cm9gq58ks002409l4fbv41qrb
slug: htb-sherlock-noted
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1743839238641/9c143ca8-fd27-4192-9a8a-ce2e8abfad67.png
tags: forensics, hackthebox, cybersecurity-1

---

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">This write-up is a part of the <a target="_blank" rel="noopener noreferrer nofollow" href="https://blog.cyberethical.me/series/htb-sherlocks" style="pointer-events: none">HTB Sherlocks series</a>. Sherlocks are investigative challenges that test defensive security skills. I encourage you to try them out if you like digital forensics, incident response, post-breach analysis and malware analysis. <a target="_self" rel="noopener noreferrer nofollow" href="https://blog.cyberethical.me/go-htbapp" style="pointer-events: none">Are you ready to start the investigation?</a></div>
</div>

# Incident Details

**Name**: [Noted](https://blog.cyberethical.me/go-htbapp)  
**Category**: DFIR  
**Difficulty**: Easy ([*Solved*](https://labs.hackthebox.com/achievement/sherlock/555018/585))

> Simon, a developer working at Forela, notified the CERT team about a note that appeared on his desktop. The note claimed that his system had been compromised and that sensitive data from Simon's workstation had been collected. The perpetrators performed data extortion on his workstation and are now threatening to release the data on the dark web unless their demands are met. Simon's workstation contained multiple sensitive files, including planned software projects, internal development plans, and application codebases. The threat intelligence team believes that the threat actor made some mistakes, but they have not found any way to contact the threat actors. The company's stakeholders are insisting that this incident be resolved and all sensitive data be recovered. They demand that under no circumstances should the data be leaked. As our junior security analyst, you have been assigned a specific type of DFIR (Digital Forensics and Incident Response) investigation in this case. The CERT lead, after triaging the workstation, has provided you with only the Notepad++ artifacts, suspecting that the attacker created the extortion note and conducted other activities with hands-on keyboard access. Your duty is to determine how the attack occurred and find a way to contact the threat actors, as they accidentally locked out their own contact information.

# Evidences

%%[support-cta] 

> All evidence files are marked as readonly right after acquiring and their hash is written down. Read-only attribute does not affect the hash of a file.

01: ZIP archive file

```plaintext
$ Noted.zip
0f4628bc37d275178158aa108db8231b15435b79d76230a620c046d9129eaef7
```

File structure after unzipping:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743797583748/3c61b36c-bade-4306-bf38-8611a670a8fd.png align="center")

```plaintext
$ config.xml
85c58374d83d1f47f089ff2fded34958d555c4fa3d2bce3fe50d60865cf05c22
$ session.xml
ebd010709f828e1239e24df5c020e84ba66d3082a523ff399c135b9d2aec96bb
$ LootAndPurge.java@2023-07-24_145332
57177080cac6e0a4ee0f10bc80587b28ac6d2fb38be417711a28b52a7a229523
$ YOU HAVE BEEN HACKED.txt@2023-07-24_150548
3700b7dc69d0f8570485bf8b5a4c4f7f84abcd722dcb7addc609129e418e932e
```

# Analysis

both pastes.io and pastebin.com contain same message.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743797597371/d2be5e69-7528-49ac-b976-63b672c336dc.png align="center")

Password protected

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743797613076/b62305c8-5cfc-4e20-a438-5250c7339a17.png align="center")

Wow that's quite a lot - is it an address of some kind of cryptocurrency market?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743797617822/06f0f3fe-5330-4803-bdf0-72f6ac152a58.png align="center")

## Questions

1. What is the full path of the script used by Simon for AWS operations?
    

`config.xml`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743797633515/a0ecc807-3362-4c2d-b383-3cc390921af9.png align="center")

2. The attacker duplicated some program code and compiled it on the system, knowing that the victim was a software engineer and had all the necessary utilities. They did this to blend into the environment and didn't bring any of their tools. This code gathered sensitive data and prepared it for exfiltration. What is the full path of the program's source file?
    

`session.xml`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743797654566/0d60a005-6bb4-434b-a069-65b6a3cf18f6.png align="center")

3. What's the name of the final archive file containing all the data to be exfiltrated?
    

`LootAndPurge.java`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743797695377/e804bebb-5a43-41ec-9b99-cdda3b47b07b.png align="center")

4. What's the timestamp in UTC when attacker last modified the program source file?
    

`session.xml`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743797720721/8eb628bc-bed0-4b85-8e92-4d1ef00f7e1d.png align="center")

[Great thread](https://community.notepad-plus-plus.org/topic/22662/need-explanation-of-a-few-session-xml-parameters-values/7) on how to decode those values.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743797729706/13d9f65f-0b2d-4bca-91ff-65176db14731.png align="center")

This one is really hard in comparison to other questions :P

```py
>>> (31047188 << 32) | (-1354503710 & 0xFFFFFFFF)
133346660033227234
```

Then use the [epoch converter](https://www.epochconverter.com/ldap) to get the right time.

5. The attacker wrote a data extortion note after exfiltrating data. What is the crypto wallet address to which attackers demanded payment?
    
6. What's the email address of the person to contact for support?
    

Both answers in the pastebin note.

# Data Recovery

Not needed.

# Lessons Learned

* How Notepad++ stores timestamps
    

# Additional readings

%%[follow-cta] 

* [NIST Computer Security Incident Handling Guide](https://www.nist.gov/privacy-framework/nist-sp-800-61)