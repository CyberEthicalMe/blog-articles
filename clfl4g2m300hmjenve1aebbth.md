---
title: "Artifacts of Dangerous Sightings"
seoTitle: "HackTheBox - Artifacts of Dangerous Sightings - Writeup"
datePublished: Thu Mar 23 2023 13:00:39 GMT+0000 (Coordinated Universal Time)
cuid: clfl4g2m300hmjenve1aebbth
slug: htb-cyberapocalypse-artifacts-of-dangerous-sightings
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1679555388949/1c103c5e-a4c5-4017-b9f0-c3a6b85031a2.png
tags: ads, hacking, ctf

---

# Introduction

> Pandora has been using her computer to uncover the secrets of the elusive relic. She has been relentlessly scouring through all the reports of its sightings. However, upon returning from a quick coffee break, her heart races as she notices the Windows Event Viewer tab open on the Security log. This is so strange! Immediately taking control of the situation she pulls out the network cable, takes a snapshot of her machine and shuts it down. She is determined to uncover who could be trying to sabotage her research, and the only way to do that is by diving deep down and following all traces ...

# Details

* Category: Forensics
    
* Difficulty: Medium
    
* Given: VHDX image file
    

%%[follow-cta] 

# Exploration

I'm mounting the image file in the Windows using the PowerShell

```powershell
Mount-DiskImage -Access ReadOnly -ImagePath C:\ws\vm\shared\2023-03-09T132449_PANDORA.vhdx
```

Taking into consideration that Pandora discovered the Security logs open with the belief that she got hacked, I'm starting with browsing the PowerShell history - and it appears to be a good guess!

```powershell
$ cat Users/Pandora/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadline/ConsoleHost_history.txt

type finpayload > C:\Windows\Tasks\ActiveSyncProvider.dll:hidden.ps1
exit
Get-WinEvent
Get-EventLog -List
wevtutil.exe cl "Windows PowerShell" 
wevtutil.exe cl Microsoft-Windows-PowerShell/Operational
Remove-EventLog -LogName "Windows PowerShell"
Remove-EventLog -LogName Microsoft-Windows-PowerShell/Operational
Remove-EventLog
```

Here it is visible that some script is being suspiciously hidden in [ADS](https://owasp.org/www-community/attacks/Windows_alternate_data_stream) of the regular Windows DLL.

%%[support-cta] 

# Analysis

<details data-node-type="hn-details-summary"><summary>Alternate Data Stream (ADS)</summary><div data-type="detailsContent">File attributes only found on the NTFS file system. Alternate data streams allow files to contain more than one stream of data. Windows Explorer doesn’t provide a way of seeing what ADS are in a file.</div></details>

We can easily view them in the Command Line

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679519086317/7fa79b75-8967-4d97-91e6-ec7991cc57cd.png align="center")

and access its content via PowerShell

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679519104236/b228b5c6-d321-4edf-8f7c-a8fb78326dec.png align="center")

This contains executable base64-encoded PowerShell command. By using a such tool as [CyberChef](https://gchq.github.io/CyberChef/) I could easily decode the text and after a two-step deobfuscation I could find a flag.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679519272454/7e4d2ee8-4c71-4eb1-bc0e-2780737503fa.png align="center")

%%[join-cta]