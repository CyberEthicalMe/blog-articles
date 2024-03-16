---
title: "Forensics: Confinement"
seoTitle: "Hack The Box CFT Cyber Apocalypse 2024 - Forensics: Confinement"
seoDescription: "Write-up for the Confinement challenge from HTB Cyber Apocalypse 2024. Malware hunting, file recovery."
datePublished: Fri Mar 15 2024 22:03:03 GMT+0000 (Coordinated Universal Time)
cuid: cltt7gkwn000e09l2byv47lt0
slug: htb-cyber-apocalypse-forensics-confinement
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1710539785969/0ab20b8b-3e78-4755-b800-45bc2029af64.png
tags: ethical-hacking, ctf-writeup

---

# Introduction

> Our clan's network has been infected by a cunning ransomware attack, encrypting irreplaceable data essential for our relentless rivalry with other factions. With no backups to fall back on, we find ourselves at the mercy of unseen adversaries, our fate uncertain. Your expertise is the beacon of hope we desperately need to unlock these encrypted files and reclaim our destiny in The Fray.  
> Note: The valuable data is stored under \\Documents\\Work

**Type**: Forensics  
**Difficulty**: Hard

# Initial recon

Given: single AD1 file.

```plaintext
$ Confinement.ad1 (read-only)
d8cc0505f4a125bb24eaa0a955bc83800678a24570daefa3ab0936a9a095a1e3
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710527686284/9d7b9cde-fcf1-42f7-a70e-b8525c037447.png align="center")

# Investigation

Before opening the file I'm marking the files as readonly and noting down its hash. File system contained in AD1 image can be browsed with a [FTK Imager](https://www.exterro.com/digital-forensics-software/ftk-imager).

<details data-node-type="hn-details-summary"><summary>Extension: *.ad1</summary><div data-type="detailsContent">AD1 (Access Data 1) is a disk image file used to hold file-level acquisitions. This format is exclusively used in the Forensic Toolkit by Accessdata.</div></details>
![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710528889702/6adc1238-9069-46bc-a1b2-66dc68c907d5.png align="center")

From the challenge description we know that:
- owner of the files was struck with a ransomware rendering the files unusable,
- important files are located under `\Documents\Work`; because this is Windows system I assume it's `%USERPROFILE%\Documents\Work`
- objective: recover file
    

## Retrieving encrypted files

Content of the important directory

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710529603240/d0b23b11-ec88-4948-8b52-ea050598bf87.png align="center")

I'm inspecting the `ULTIMATIM.hta` file using the FTK Viewer preview panel. It looks like a ransomware note.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710529816756/6c0ab892-ed3a-4d0d-97e4-b15c87e0effb.png align="center")

The only thing that is worth noting down is the Faction ID that could be used later.

```plaintext
Faction ID = 5K7X7E6X7V2D6F
```

I extract the `Applicants_info.xlsx.korp` file.

```plaintext
Applicants_info.xlsx.korp (read-only)
316954ea99040a2e6b9aef47d230063c1f03e4515e1552f91304dd0769d8385e
```

> In the meantime I do explore other files in the directory, but these are not relevant for us right now, so I am not elaborating on this one. I did also check hashes for other ransomware notes to make sure I did not falsly assume that all are the same - but they are the same.

Because the hex preview in the FTK Viewer really looked like some mangled bytes, I have no hopes that `strings` won't find anything of value, and I am not mistaken.

Right now I don't have tools to restore the file.

## Malware hunting

Let's backtrack a bit. We are looking at the end of the process and we have to look for the evidences of previous steps that most certainly had occurence. So what are these steps? These vary from the source to source, but let's take the core ones:

1. Infection.
    
2. Execution - data exfiltration, data encryption
    
3. Spreading.
    
4. Persistence.
    
5. Clean up.
    

So at this point we are looking for any traces of

* unknown files (in `Downloads`, `Desktop`, `c:\temp`, etc), that includes especially executable files (DLLs and EXEs)
    
* suspicious connections (HTTP over port 80, `nc` uses)
    
* any PowerShell executions (regular users rarely if ever use PowerShell; system rarely)
    
* anything really that outstands
    

### Reverse shell

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710533728550/aba07833-871d-4182-a02c-2ffa1ab37dcc.png align="center")

This one is a suspicious file and should not be there - without doubt. Unfortunately, it is of no use in this scenario.

> ATS\_setup.. [I wonder what could that be](https://youtu.be/RWuxRCrt0gs?t=26)..

### Windows Timeline

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710534527139/4dca4829-bd7a-4284-946e-79313cd2fba0.png align="center")

Using famous Eric Zimmerman's [Windows 10 Timeline database parser](https://ericzimmerman.github.io/#!index.md) allowed me to browse through the `ActivitiesCache.db` - nothing paticulary interesting (`7z2107-x64.exe`, `ftk imager.exe`, `treesizefree.exe`).

### Windows Defender

Finally, I started analyzing the Windows Defender logs (`C:\ProgramData\Microsoft\Windows Defender\Support\`). Here are the findings:

* (`HackTool:Win32/Mimikatz!pz`) `C:\Users\tommyxiaomi\Documents\mimikatz.exe`
    
* (`Trojan:Win32/CryptInjecp`) `C:\Users\tommyxiaomi\Documents\fscan64.exe`
    
* (`Trojan:Win32/Wacatac.B!mp`) `C:\Users\tommyxiaomi\Documents\intel.exe`
    
* (`Trojan:Win32/MpTamperDisableFeatureWd.p`) `C:\Windows\System32\Dism.exe /online /Disable-Feature /FeatureName:Windows-Defender /Remove /NoRestart /quiet`
    
* (`Trojan:Win32/MpTamperDisableFeatureWd.p`)
    
* (`VirTool:Win32/DefenderTamperingRestore`) `regkeyvalue hklm\software\microsoft\windows defender\spynet\DisableBlockAtFirstSeen`
    
* (`HackTool:Win32/LaZagne`) `C:\Users\tommyxiaomi\Documents\browser-pw-decrypt.exe`
    
* (SAM dumping) `C:\Windows\System32\cmd.exe(cmd.exe /c reg.exe save hklm\sam C:\Users\TOMMYX~1\AppData\Local\Temp\crkrbigyfjqk)`
    

<details data-node-type="hn-details-summary"><summary>Security Account Manager</summary><div data-type="detailsContent">SAM (Security Account Manager) is a database file that stores users' passwords. The user passwords are stored in a hashed format in a registry hive either as an LM hash or as an NTLM hash. This file can be found in <code>%SystemRoot%/system32/config/SAM</code> and is mounted on <code>HKLM/SAM</code> and <code>SYSTEM</code> privileges are required to view it.</div></details>

Almost all of them are self explainatory - apart from `intel.exe`. Could it be the actual crypter? Unfortunately all of the above executables can no longer be found at their destination. So can we somehow obtain the files? We don't have `*.pcap` file to grab the binary stream, we don't have access to any memory dumps for more binary dumping from RAM.

But there is another place we can look for this file.

![Bingo](https://cdn.hashnode.com/res/hashnode/image/upload/v1710536984118/9d978e67-b66c-4be7-92ca-e4d730c0ea3b.png align="center")

Bingo.

### Windows Defender Quarantine

We can't access the binaries becasue of how they are stored in the quarantine.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710537153078/ae0e5c40-8368-456c-9201-06b809cc1885.png align="center")

So for that I'm using a [Python script](https://github.com/knez/defender-dump)`defender-dump.py` after I've mounted the AD1 image with the FTK Imager.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710537327881/f08f133c-8b6b-45bf-9fd2-37768f728061.png align="center")

> When perfoming this operation on Windows - under no circumstances turn the Windows Defender off. Use the directory exclusion because they will keep being qurantined :)

## Crypter (`intel.exe`)

Let's analyze what it is doing (`dotPeek`).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710537509221/149f850a-2302-4b6f-a437-2cd117e8c2fc.png align="center")

1. CTF Safeguard (do not run on the CTF participant system if mistakendly run by one).
    
2. Generate user/faction ID (it is visible on the `ULTIMATE.hta`).
    
3. Create CoreEncrypter object.
    
4. Iterate recursively over all directory entries and run the encryption on files, unless they don't qualify for encryption.
    

At this point we are ready to write decryptor. "Missing" part is a Faction ID we have obtained from the ransom note.

```csharp
using System.Text;
using System.Security.Cryptography;

const string salt = "0f5264038205edfb1ac05fbb0e8c5e94";
const string UID = "5K7X7E6X7V2D6F";

var filePath = "Applicants_info.xlsx.korp";
var password = GetHashCode(UID, salt);

DecryptFile(filePath, password);

void DecryptFile(string encodedFilePath, string password)
{
    if (!encodedFilePath.EndsWith(".korp"))
    {
        throw new ArgumentException(encodedFilePath);
    }

    byte[] buffer = new byte[ushort.MaxValue];
    var rfc2898DeriveBytes = new Rfc2898DeriveBytes(password, new byte[8] { 0, 1, 1, 0, 1, 1, 0, 0 }, 4953);
    var rijndaelManaged = new RijndaelManaged();
    rijndaelManaged.Key = rfc2898DeriveBytes.GetBytes(rijndaelManaged.KeySize / 8);
    rijndaelManaged.Mode = CipherMode.CBC;
    rijndaelManaged.Padding = PaddingMode.ISO10126;
    rijndaelManaged.IV = rfc2898DeriveBytes.GetBytes(rijndaelManaged.BlockSize / 8);

    using (var encodedFileStream = new FileStream(encodedFilePath, FileMode.Open, FileAccess.Read))
    {
        var decryptedFilePath = encodedFilePath.Replace(".korp", string.Empty);
        using (var decryptedFileStream = new FileStream(decryptedFilePath, FileMode.Create, FileAccess.Write))
        using (var cryptoStream = new CryptoStream(decryptedFileStream, rijndaelManaged.CreateDecryptor(), CryptoStreamMode.Write))
        {
            int count;
            do
            {
                count = encodedFileStream.Read(buffer, 0, buffer.Length);
                if (count != 0)
                    cryptoStream.Write(buffer, 0, count);
            }
            while (count != 0);
        }
    }
}

string GetHashCode(string pass, string salt)
{
    var password = pass + salt;
    using (var cryptoServiceProvider = new SHA512CryptoServiceProvider())
    {
        byte[] bytes = Encoding.UTF8.GetBytes(password);
        return Convert.ToBase64String(cryptoServiceProvider.ComputeHash(bytes));
    }
}
```

Running the code against the encrypted XLSX file yields restored file. Because I don't want to deal with the potential malware in the XLSX file - I unzip the file and uses Visual Studio Code search functionality to grab the flag.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710538511891/c33662a1-9f84-4713-8886-8602a1385d69.png align="center")