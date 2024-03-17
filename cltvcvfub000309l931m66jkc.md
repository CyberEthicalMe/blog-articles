---
title: "Forensics: Oblique Final"
seoTitle: "Hack The Box CFT Cyber Apocalypse 2024 - Forensics: Oblique Final"
seoDescription: "To be honest, it's great for the challenges and fun, but that's really scary that such things exists. Are you ready to run with the wolves? Write-up."
datePublished: Sun Mar 17 2024 10:10:07 GMT+0000 (Coordinated Universal Time)
cuid: cltvcvfub000309l931m66jkc
slug: htb-cyber-apocalypse-forensics-oblique-final
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1710669066432/9545d487-e55b-415f-84a8-dc1eb52f6ba8.png
tags: reverse-engineering, forensics, ethical-hacking, ctf-writeup

---

# Introduction

> As the days for the final round of the game, draw near, rumors are beginning to spread that one faction in particular has rigged the final! In the meeting with your team, you discuss that if the game is indeed rigged, then there can be no victory here... Suddenly, one team player barged in carrying a Windows Laptop and said they found it in some back room in the game Architects' building just after the faction had left! As soon as you open it up it turns off due to a low battery! You explain to the rest of your team that the Legionnaires despise anything unethical and if you expose them and go to them with your evidence in hand, then you surely end up being their favorite up-and-coming faction. "Are you Ready To Run with the Wolves?!"

**Type**: Forensics  
**Difficulty**: Insane  
**Event**: [**Hack The Box Cyber Apocalypse 2024: Hacker Royale**](https://ctf.hackthebox.com/event/details/cyber-apocalypse-2024-hacker-royale-1386) ([**ctftime**](https://ctftime.org/event/2255))

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">I will use your initial attention to express my gratitute towards the author of this challenge as it was for me same eye opener as when I discovered the imaginary numbers like 20 years ago, AKA: things that do not exist in your universe until they are revealed to you. To be honest, it's great for the challenges and fun, but that's <em>really scary</em> that such things exists. And for these kind of learning and emotions I love forensics challenges. So thank you <strong>c4n0pus</strong>🙇‍♂️.</div>
</div>

# Initial recon

Given: Windows hibernation file.

```plaintext
$ hiberfil.sys (read-only)
f9f96147bfe041e3174eb988e4fc10e16c17a82b20bb6b7f9aff23126b523699
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710596726348/ef2c77d4-985c-4e16-86df-161294298c1c.png align="center")

# Investigation

<details data-node-type="hn-details-summary"><summary>Windows Hibernation File</summary><div data-type="detailsContent"><code>hiberfil.sys</code> is a hidden system file is located in the root folder of the drive where the operating system is installed. The Windows Kernel Power Manager reserves this file when you install Windows. The size of this file is approximately equal to how much random access memory (RAM) is installed on the computer. The computer uses the <code>hiberfil.sys</code> file to store a <strong>copy of the system memory</strong> on the hard disk when the hybrid sleep setting is turned on. If this file is not present, the computer cannot hibernate.</div></details>

Learned by experience, whenever I'm dealing with the memory dumps I start with [The Volatility Framework](https://volatilityfoundation.org/the-volatility-framework/). Unfortunately my hopes were quickly vanished becasue `volatility` can't operate directly on the `hiberfil.sys`. I scratched my head becasue it seemed unlikely that noone even thought about adding support for the hibernation files to the such profound tool.

Luckily for all of us, as of the time this CTF event started, [ForensicXlab](https://www.forensicxlab.com/about/) already did the work and created a [pull request](https://github.com/volatilityfoundation/volatility3/pull/1036) for `volatility3` adding two new plugins: `windows.hibernation.Info` and `windows.hibernation.Dump`.

> Seeing how long it took someone to finally come up with the plugins that easen working with hibernation memory dumps - 👏 for the [k1nd0ne](https://infosec.exchange/@k1nd0ne), arigato!

After switching to the PR branch, my `volatility` was ready to create the `memory_layer.raw` file so I can finaly start analysis.

> For the sake of simplicity I'm using `vol3` alias for `python c:/ws/tools/volatility3/vol.py`

```plaintext
$ git fetch origin pull/1036/head:feature/hibernation-layer
$ vol3 -f hiberfil.sys windows.hibernation.Dump --version 0
```

> Why am I using version `0`? I just assumed that memory dump originates from Windows 10, and wanted to start from the latest possible
> 
> * 0 \[Windows 10 1703 to Windows 11 23H2\]
>     
> * 1 \[Windows 8/8.1\]
>     
> * 2 \[Windows 10 1507 to 1511\]
>     
> * 3 \[Windows 10 1607\]
>     

```plaintext
$ memory_layer.raw (read-only)
0cfb69681ee202f8f65245301ec113f6ea5fb1546f8bac0279baf65d8a800bc0
```

## Analyzing memory dump

Now it's the most interesting and time-taking phase. I dump and organize various outputs from `volatility` simultanously trying to note down anything that could be analized later in the greater details. This was the first time when I was solving the Insane difficulty challenge, so my choice of plugins expanded to the ones I've never tried - becasue I really couldn't pinpoint what else could I do with what I have at my disposal. But about this a bit later.

I have used following plugins (some of them targeted on the specific processes):

```yaml
- windows.pslist.out
- windows.dumpfiles.DumpFiles
- windows.cmdline.CmdLine.out
- windows.envars.Envars.6348.out
- windows.filescan.FileScan.out
- windows.mftscan.MFTScan.out
- windows.mbrscan.MBRScan.out

- windows.netscan.NetScan.out
- windows.netstat.NetStat.out
- windows.dlllist.DllList.out
- windows.handles.Handles.6348.out
- windows.handles.Handles.out
- windows.modscan.ModScan.out
- windows.modules.Modules.out
- windows.privileges.Privs.6348.out
- windows.registry.hivelist.HiveList.out
- windows.registry.hivelist.HiveScan.out
- windows.vadinfo.VadInfo.out

- windows.strings.Strings
```

Top ones are what usually was enough for me to find the malicious process and dump it so I can analyze further, discover shell run command or suspicious environment variable. In the middle are ones that I run depending on the context finshing with these I've never touched. `windows.strings.Strings` is a really "I don't know what I am doing, just grab me all strings from that file and I will figure it out later when you finish, in like... 4 or 5 hours from now".

Here is my collection

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710599440362/baec784e-1bdf-4aa9-b84c-bccc285e7182.png align="center")

As you can probably see one of the process cought my attention and will forever have special place in my heart 😅 .

## Process #6348

| PID | PPID | Offset(V) | ImageFileName |
| --- | --- | --- | --- |
| 6560 | 5852 | 0xac8d5fba2080 | explorer.exe |
| 6348 | 6560 | 0xac8d5ef4e080 | TheGame.exe |
| 2460 | 6348 | 0xac8d5f8e2080 | conhost.exe |

`TheGame.exe` is not usual executable you find on Windows system. Especialy one residing in this location (thanks to `windows.cmdline.CmdLine`):

```bash
6348    TheGame.exe     "C:\Users\architect\Desktop\publish\TheGame.exe"
```

Same executable appear in the list of connections

```markdown
Offset	        Proto	LocalAddr	LocalPort	ForeignAddr	ForeignPort	State	PID	    Owner	    Created
0xac8d5f6dd8a0	TCPv4	10.0.2.15	49829	    20.12.23.50	443	        CLOSED	6348	TheGame.exe	2024-02-08 22:43:14.000000
```

..but again, *cul*\-*de*\-*sac*. It is easy to establish that `20.12.23.50` belongs to [Windows Update service](https://www.netify.ai/resources/ips/20.12.23.50). Chasing answer if it is true, is not my concern especially because connection was established over SSL - and that's definitely not a common thing to put into CTF challenge.

Anyway, I continue with dumping the binary from the process.

```bash
$ vol3 -f memory_layer.raw windows.pslist.PsList --pid 6348 --dump
```

Hmm, wrong file?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710605686798/490e6927-3ae2-4f5e-a000-41e506bc3809.png align="center")

Maybe the binary is incomplete/malformed. Let's see if there are other places from which we can get the binary.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710605994460/46a8e2b6-cebe-4e7b-b0c3-ff35cf91463f.png align="center")

After checking all files it becomes clear that we won't get anything apart from one file. I proceed with decompiling `TheGame.dll` in [ILSpy](https://github.com/icsharpcode/ILSpy).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710604268599/b9176d6b-e575-4dfa-8018-1ebf21235a6f.png align="center")

Let's have a look at the `Main(..)` method.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710604420851/ed0420a5-bdbc-4026-89af-21c784d9ff67.png align="center")

1. Try loading list of DLLs.
    
2. Initilize AMSI engine.
    
3. Tests the engine using hardcoded binary data.
    
4. Load `kernel32.dll` test it with AMSI.
    
5. If AMSI does not report file as malicious, tries injecting DLL with the same binary data that raises AMSI alert.
    
6. Goes limbo with **many**`Nop` commands.
    

Ok, a lot is happening there so let's start with some definitions.

<details data-node-type="hn-details-summary"><summary>Antimalware Scan Interface (AMSI)</summary><div data-type="detailsContent">Versatile interface standard that allows the applications and services to integrate with any antimalware product that's present on a machine.</div></details><details data-node-type="hn-details-summary"><summary>No-operation instruction (nop, no-op)</summary><div data-type="detailsContent">Machine language instruction and its assembly language mnemonic, programming language statement, or computer protocol command that does <strong>nothing</strong>.</div></details>

Although I suspect what that base64 encoded binary data [is](https://www.eicar.org/download-anti-malware-testfile/), let's throw it into [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)&input=V0RWUElWQWxRRUZRV3pSY1VGcFlOVFFvVUY0cE4wTkRLVGQ5SkVWSlEwRlNMVk5VUVU1RVFWSkVMVUZPVkVsV1NWSlZVeTFVUlZOVUxVWkpURVVoSkVnclNDb05DZz09).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710605259638/b961053c-f8a6-40a5-8aed-5e3827a5f80e.png align="center")

<details data-node-type="hn-details-summary"><summary>EICAR Anti-Virus Test File</summary><div data-type="detailsContent">Test file that was developed by the European Institute for Computer Antivirus Research (EICAR) and Computer Antivirus Research Organization (CARO), to test the response of computer antivirus programs. Instead of using real malware, which could cause real damage, this test file allows people to test anti-virus software without having to use a real computer virus.</div></details>

Okey, let's think for a moment.

In the `main(..)` method of a `TheGame.dll` .NET library there are 8 more DLLs loaded, do nothing with them, then proceed to load some AMSI module with a purpose of.. **writing some bytes to the core Windows library**.. when for sure it is already in use! So that won't gonna happen - that's why the exception is swollen in `try/catch` clause.

> Ok now I will start talking like it is just an another CTF challenge, another Mr. Know-it-all. But reaching from this point to the moment when I realized what I have to do, took me **4 (four) days**. To be more precise something around 20-25hrs but because of what is going to happen is probably why this challenge is rated as Insane.

It's either a rabbit hole that someone put some effort to setup, or there is something that I am missing from the picture.

*You explain to the rest of your team that the Legionnaires despise anything unethical and if you expose them and go to them with your evidence in hand, then you surely end up being their favorite up-and-coming faction. "Are you Ready To Run with the Wolves?!"*

Why are the `Ready To Run` and `Wolves` capitalized?

> ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710607052298/78686b6b-e389-41eb-9199-485ff4108e52.png align="center")
> 
> ..i don't think this is what the author wants me to search for... but for sure **I** can recommend this:
> 
> %[https://open.spotify.com/track/6r6n5OmXkdsOxN2iCj5UE0?si=82cfd1ed1bba4419] 

Oh you sneaky s..

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710607739242/ace6970c-1162-4c32-bd79-82d9be5ba57a.png align="center")

# Ready to run with the wolves

%%[support-cta] 

I'm in no competence to explain what ReadyToRun Deployment, Ahead-Of-Time Compilation are and how important they are not only for this challenge but for all `dotnet` developers and security researchers. That's why I am recommending to read great articles and resources I've gathered in the last section as well as other write-ups for this challenge. And for now let me explain this concept in the Minimum-Valuable-Product way.

## R2R and AOT

Step back. `Source Code -> IL -> Native`. CIL programming language code is translated to the Common Intermidiate Language which then Common Language Runtime executes by performing Just-in-time Compilation, compiling CIL to native instructions. This happens this way to deliver versatility for CIL languages - software can run on different platforms, not only on that on which source code was compiled (because each platform has its CLR).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710609960036/7bd355cf-55bc-4cec-82e7-49b63d2ed6d1.png align="center")

*(source:* [*https://medium.com/@kunaltandon.kt/c-clr-il-jit-compilation-code-access-security-explained-269124121f5*](https://medium.com/@kunaltandon.kt/c-clr-il-jit-compilation-code-access-security-explained-269124121f5)*)*

Ahead-Of-Time was a term existing for some time now, but R2R was introduced with the release of [.NET Core 3.0](https://documentation.red-gate.com/sa8/building-your-assembly/using-smartassembly-with-readytorun-images-net-core-3). If you are familiar with the term [Template Specialization](https://www.geeksforgeeks.org/template-specialization-c/) (or similar) - you are golden. At least that's how I understand whats happening.

So basicaly, you have your source code translated to CIL, generic code. But instead of waiting for your software to run on specific platform and let CLR to compile your CIL into native machine instructions - you just compile it ***ahead of time***. Then you include that compiled, native instructions into your software so it is ***ready to run*** on that platform.

## R2R Stomping

Ok, so how does it relate to our `TheGame.dll`? See, when decompiler inspects library or executable it just **by default** run through the IL and extrapolates on what the software is doing. But when such software runs, it executed the native instructions - provided both from CLR and AOT Compilation.

What if.. someone compiles the DLL with R2R targeted specific architecture - then for example injects own native instructions overridding the AOT compilation. Assuming there is just enough space in R2R portion of assembly to hold new payload, that one will end up having a single software copy that can behave very differently from how it was coded!

> I encourage at least glancing throught the R2R Stomping article for more detailed presentation.

This, my ladies and gents - is the reason I was loosing my mind over this one challenge for a couple last days.

## Detecting R2R

Actually there are many signs that we are dealing with R2R assembly - I just didn't know about them 😉

dotPeek:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710626144665/7d793349-b563-45c7-85d7-43844e8645d0.png align="center")

dnSpy:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710663231309/6789e898-b950-4563-afc1-aa0ea103fb04.png align="center")

> R2R File Format specifies that (quote from the linked GitHub document):
> 
> * The PE file is always platform specific
>     
> * CLI Header Flags field has set `COMIMAGE_FLAGS_IL_LIBRARY` (0x00000004) bit set
>     
> * CLI Header `ManagedNativeHeader` points to READYTORUN\_HEADER
>     

And finally, if you are not a fan of manual/visual analysis - here comes the quickest way to check if you are dealing with the R2R assembly - rule for the ultimate [Yara](https://virustotal.github.io/yara/).

```c
import "pe"
// source: 
// https://research.checkpoint.com/2023/r2r-stomping-are-you-ready-to-run/

rule r2r_assembly
{
    meta:
        author = "jiriv"
        description = "Detects dotnet binary compiled as ReadyToRun - form of ahead-of-time (AOT) compilation"
    condition:
        // check if valid PE
        uint16(0) == 0x5a4d and uint16(uint32(0x3c)) == 0x4550 and
        // check if dotnet -> .NET Directory is present
        pe.data_directories[pe.IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR].virtual_address != 0 and
        // check if ManagedNativeHeader exists -> ManagedNativeHeader RVA is not 0 inside .NET Directory
        uint32(pe.rva_to_offset(pe.data_directories[pe.IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR].virtual_address) + 0x40) != 0 and
        // check if it is R2R -> RTR magic signature is present (0x00525452 == "RTR" in ascii)
        uint32(pe.rva_to_offset(uint32(pe.rva_to_offset(pe.data_directories[pe.IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR].virtual_address) + 0x40))) == 0x00525452
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710663580488/4a832138-ca27-46f1-948d-cfd3006e264b.png align="center")

# Extracting shellcode

<details data-node-type="hn-details-summary"><summary>Shellcode</summary><div data-type="detailsContent">Small piece of executable code used as a payload, built to exploit vulnerabilities in a system or carry out malicious commands. Shellcode is commonly written as assembly instructions.</div></details>

Now what we know what `TheGame.dll` really is, lets see the `Main(..)` method once again, but from different perspective. Load the assembly into ILSpy, right click on the `Main(..)` and choose "Decompile to new tab". We can stack two of these horizontally so we can have easier time to see the difference.  
In one of the tabs select "ReadyToRun" (to be honest, I've never been aware that this dropdown exists..).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710664262683/20244770-65f9-476f-8d58-fceec7dd3765.png align="center")

I'm not a assembly guy, but what can be seen in ReadyToRun view, is nothing about loading some DLLs and checks. There are a lot of `mov` instructions and no binary loading, no jumps, etc.

At the top of the file/view first instruction is at `0x1DB0` . The moment when shellcode ends is a moment when similarity between IL view and R2R view starts - `0x33B0`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710665117918/c24751a9-bf8f-468f-8613-c6f0f7518526.png align="center")

Let's then carve it out then directly from the file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710665595495/6f47232c-e8fe-40b4-98e7-b3abf5a6eaaf.png align="center")

```plaintext
$ shellcode.bin (read-only)
acb6c622f2ff6dafdb1391561b26d2abaa3944e342510c6ed2afcc4231cc0680
```

# Shellcode analysis

Because trying to run the shellcode directly on a live system would be of the same level of responsibility as running the `Invoke-Expression` or `eval` copied directly from the malware (not really responsible and smart) - there are some solutions that enables **shellcode emulation**.

> To be honest it's not even that easy to run or analyze such shellcode, well, because these are just the plain assembly instructions - no PE headers so it is not recognized as "a thing" in dissasemble tools, nor `strings` won't help in our situation, `file` command returns just "data".

I'm going to use [speakeasy](https://github.com/mandiant/speakeasy) Python module for that purpose.

> Quick tip: after git cloning the speakeasy repository and installing requirements, if you would like to call `speakeasy` regardless what you current working directory add following to your shell profile (`.bashrc`, `.zshrc`) with the correct path of course.  
> `export PYTHONPATH="$PYTHONPATH:/opt/speakeasy"`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710667961505/ca056181-5de8-4d73-b03a-db1161fdcfde.png align="center")

Voilà! It runs, but apparently stops after it verifies that current host is not "ARCH\_WORKSTATION-7". Let's change it, shall we?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710668463459/82b8dbe7-f2b6-485f-b3d4-1a929911b40c.png align="center")

Finally 🎉

# Additional readings

%%[follow-cta] 

* ⭐ [Official HTB write-up](https://github.com/hackthebox/cyber-apocalypse-2024/tree/main/forensics/%5BInsane%5D%20Oblique%20Final)
    
* ⭐ [R2R Stomping – are you ready to run?](https://research.checkpoint.com/2023/r2r-stomping-are-you-ready-to-run/)
    
* ⭐ [GitHub/dotnet/runtime: ReadyToRun File Format](https://github.com/dotnet/runtime/blob/main/docs/design/coreclr/botr/readytorun-format.md)
    
* [Emulation of Malicious Shellcode With Speakeasy](https://www.mandiant.com/resources/blog/emulation-of-malicious-shellcode-with-speakeasy)
    
* [Using SmartAssembly with ReadyToRun images (.NET Core 3)](https://documentation.red-gate.com/sa8/building-your-assembly/using-smartassembly-with-readytorun-images-net-core-3)
    
* [Microsoft: ReadyToRun Compilation](https://learn.microsoft.com/en-us/dotnet/core/deploying/ready-to-run)
    
* [Conversation about crossgen2](https://devblogs.microsoft.com/dotnet/conversation-about-crossgen2/)
    
* [POOF write-up from HackTheBoo 2022 using volatility2](https://blog.cyberethical.me/hacktheboo-2022-htb-ctf-write-ups#heading-poof)
    
* [volatility3 pull request with the hibernation plugins by ForensicXlab](https://github.com/volatilityfoundation/volatility3/pull/1036)
    
* [Volatility3: Modern Windows Hibernation file analysis by ForensiXlab](https://www.forensicxlab.com/posts/hibernation/)