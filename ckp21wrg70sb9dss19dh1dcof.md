## My Hacking Lab Setup

*Here is my way of learning ethical hacking and cybersecurity; this list is not at all comprehensive. I will be coming back to this article and update its content. Please let me know in the comments below if you found something new or if I can improve/add something to my repertoire.*

> 🔔 `CyberEthical.Me` is maintained purely from your donations - if you would like to boost the community, consider one-time sponsoring at the 🍻 [Buymeacoffee](https://www.buymeacoffee.com/asentinn) or use the [Sponsor](https://blog.cyberethical.me/sponsor) button. Cheers!

%%[bmac-button]

***
# Contents
1. [Setup Overview 🔌](#setup-overview)
2. [Kali 💀](#kali)
3. [Useful sites 🌐](#useful-sites)
4. [Theory Courses 📚](#theory-courses)
5. [Practical Learning 🔥](#practical-learning)
6. [Content Creation 📣](#content-creation)
7. [Time Tracking ⏰](#time-tracking)
8. [Set Your Goal 🏁](#set-your-goal)
***

# Setup Overview 🔌

![virtualbox.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621778288363/QY3zm6gj6.png)

I'm using the Oracle Virtual Box both for my target boxes and the attack box. All boxes are running on a separate internal network and Kali box is NATed. A separate network without Internet connection is crucial, especially when planning on booting up the Metasploitable.

I don't have any Guest Additions Tools installed, as this just increasing the attack surface. I know, this is painful to get used to, but it is a conscious decision to undergo such inconvenience from time to time to keep my data more secured.

## Windows Boxes

I have created these during the [Pluralsight course](https://app.pluralsight.com/paths/certificate/ethical-hacking-ceh-prep-2018) but never used them.

All boxes have the same user/password setup and turned off some security measures, like firewall and password policy. Also, each box have a snapshot called "Fresh" that is taken after initial setup was completed. None of them is exposed to the Internet.

## Attack Box

I have chosen Kali because during my Computer Forensics course we were using Backtrack; and Backtrack got renamed to Kali in 2013. 

I create a snapshot every day, keeping only the last 5-7 of them - I never know when I make a mistake and run some malicious code or a conflicting update. 

Additional learning materials:
* [Learning Kali Linux @ LinkedIn Learning](https://www.linkedin.com/learning/learning-kali-linux-2)
* [Ethical Hacking: Understanding Ethical Hacking @ Pluralsight](https://app.pluralsight.com/library/courses/ethical-hacking-understanding)
* [Kali Linux Revealed](https://kali.training/)

## Metasploitable 2

![metasploitable.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621779839442/2o2NtpA2x.png)

Purposely vulnerably Linux server. Read more on the [official site](https://docs.rapid7.com/metasploit/metasploitable-2). 

### How to safely exploit

Snapshot Kali. Make sure Attack Box is disconnected from the Internet and connected to the internal hacknet. Turn both VMs. Perform actions. Optional: reconnect attack box to upload notes. Shutdown machines. Restore snapshots.

## TinyCore Linux
 
I don't know what it is for, but it was presented on one of the courses, so I've got it :)

# Kali 💀

This is my Attack Box Operating System. I keep here all my write-ups, articles and other content I store during the learning process.

## Terminal

![terminator.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621779819578/dDn7_y7yr.png)

I found [John Hammond](https://www.youtube.com/channel/UCVeW9qkBjo3zosnqUbG7CFw) using it on his videos and I liked the idea of [working in many terminals](https://terminator-gtk3.readthedocs.io/en/latest/) that perfectly fills the screen.

![terminator_profile.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621779870813/Vxyd5TFwF.png)

* keybindings for close terminal, open new terminal horizontal/vertically, switch to next terminal
* full screen, no title bar, no scroll bar
* infinite scroll
* transparent background (useful when referencing something I have opened in the browser)

I got a few profiles created, that differ only with font and background color, so I can quickly recognize what's happening.

## Shell aliases

```sh
$ cat ~/.shell_aliases 

alias ls='ls --color=always -hla'
alias df='df -h'
alias cls='clear'

function apt-updater {
	sudo apt-get update &&
	sudo apt-get dist-upgrade -y &&
	sudo apt-get autoremove -y &&
	sudo apt-get autoclean &&
	sudo apt-get clean
}  
```

From top to bottom:

* `ls` lists all items (for hidden entries), presents as a list with human friendly size format; `--color=always` to keep useful directory/links/etc colors;
* `df` shows disk space in human friendly format;
* `cls` - because less typing is better

Last one is a function that I am using to update the software. I try to execute it almost each time I open the system and definitely before some bigger project. 

> Just create a snapshot before firing up the upgrades.

## OOTB Applications

Software that is installed on my Kali by default.

### nmap

Obligatory to find out open ports and check for running services on the network.

### gobuster

CLI version of a DirBuster, incredibly useful to enumerate directories and files on the website.

### nikto

More advanced website scanner, apart from directory enumeration it also performs vulnerability checks.

### burpsuite

![burpsuite.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621780151475/uGsx0upo9.png)

Time spent with this tool is not a wasted time because you will use it through all your _mastery levels_. Numerous possibilities that come from intercepting and modifying the requests and responses between host and website.

### netcat

Primary choice when I need a reverse shell setup.

### searchsploit

CLI [tool](https://www.exploit-db.com/searchsploit) for searching the Exploit-DB. You can also download the exploits or PoC directly through this tool.

### hashcat

What to say more - it cracks passwords hashes.

### john

Same category like hashcat, but more powerful. I don't feel comfortable with it for now.

## CherryTree

👉 https://www.giuspen.com/cherrytree/

Planning to learn and see if it fits me better than Sublime.

## Additional downloads

### Docker

There are many examples when Docker could come handy. For example, many vulnerable applications can be launched via dockerfile, without tedious setup.

### ngrok

Useful little application for tunneling. One of the ways I'm using it is demonstrated in my [linPEAS analysis](https://blog.cyberethical.me/linpeas).

### linPEAS

"Linux local Privilege Escalation Awesome Script (linPEAS) is a script that search for possible paths to escalate privileges on Linux/Unix host". Read more about it in my [linPEAS analysis](https://blog.cyberethical.me/linpeas).

### Sublime

Quick, fast - I got used to it.

### Flameshot

Alternative for Windows Lightshot.

### pandoc

Easily convert *.md files to HTML and PDF.

### Seclist

👉 https://github.com/danielmiessler/SecLists

```sh
$ ls /usr/share/seclists

total 56K
drwxr-xr-x  11 root root 4.0K Apr 26 01:11 .
drwxr-xr-x 370 root root  12K May 23 10:33 ..
drwxr-xr-x   9 root root 4.0K Apr 26 01:09 Discovery
drwxr-xr-x   8 root root 4.0K Apr 26 01:11 Fuzzing
drwxr-xr-x   2 root root 4.0K Apr 26 01:11 IOCs
drwxr-xr-x   5 root root 4.0K Apr 26 01:11 Miscellaneous
drwxr-xr-x  12 root root 4.0K Apr 26 01:11 Passwords
drwxr-xr-x   3 root root 4.0K Apr 26 01:11 Pattern-Matching
drwxr-xr-x   9 root root 4.0K Apr 26 01:11 Payloads
-rw-r--r--   1 root root 2.0K Feb 11 22:59 README.md
drwxr-xr-x   4 root root 4.0K Apr 26 01:11 Usernames
drwxr-xr-x   9 root root 4.0K Apr 26 01:11 Web-Shells
```

_Collection of multiple types of lists used during security assessments, collected in one place. List types include usernames, passwords, URLs, sensitive data patterns, fuzzing payloads, web shells, and many more._

# Useful sites 🌐

* [Hack Me](https://hack.me/s/)
* [PentesterLab](https://pentesterlab.com/exercises)
* [CTF Challenge](https://ctfchallenge.com/)

* [CrackStation](https://crackstation.net/)
* [CyberChef](https://gchq.github.io/CyberChef/)

# Theory Courses 📚

* [Pluralsight 
Ethical Hacking (CEH Prep 2018)](https://app.pluralsight.com/paths/certificate/ethical-hacking-ceh-prep-2018)
* [LinkedIn Learning - Become an Ethical Hacker](https://www.linkedin.com/learning/paths/become-an-ethical-hacker)
* [Web Security Academy](https://portswigger.net/web-security)
* [Kontra - OWASP Top 10 for Web](https://application.security/free/owasp-top-10/)

Both platforms are paid ones, but if you are the participant of Visual Studio Dev Essentials program, you should have some free months to use on them - if you happen to work in a company that is a Microsoft Partner, ask your employee if you have access to the Microsoft Subscription benefits.

# Practical Learning 🔥

## Hack The Box

👉 https://academy.hackthebox.eu
👉 https://app.hackthebox.eu

Great materials, comprehensive and well-prepared. There are few free modules, for the more advanced you have to pay in Cubes (currency acquired through subscription or direct payment). HTB also often organizes CTFs and events together with other partners. There are also many hacking boxes put on rotations (active/retired) on their App portal (retired ones are only for subscribers).

## Try Hack Me

👉 https://tryhackme.com/

Similar to HTB, but less active on social media. They've got an easier subscription model (you are paying monthly/yearly, and you have access to every room).

## Hacker 101

👉 https://ctf.hacker101.com/ctf

Apart from being a platform for bug bounties, they have also ~20 quite interesting CTF rooms. It operates on the similar fashion as Juice Box, where you are getting flags for performing actions, instead of hunting for the plaintext flag. 

> Unfortunately you won't see the writups from Hacker 101 on my site becasue they didn't want the solutions to be available online.

# Content Creation 📣

![Screen-Shot-2019-01-28-at-11.38.16-AM-1024x575.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1621780830717/AJ3IhZSeW.png)

_Sourced from [the KLM Group](https://theklmgrp.com/the-learning-pyramid-a-model-that-reflects-a-range-of-methods-for-facilitating-a-conversation/)_

Time for some serious advices. Start a blog. Launch a Youtube channel. Twit, post on Instagram and Facebook. Join Community Groups on Discord.

Just remember one thing - don't lose your goal. Your goal is to learn hacking and cybersecurity, not to become a celebrity. But in the same time try to engage your community - even when you don't have any (like me right now :D ). Act like you do, ask questions, participate in discussions. First - it will become natural for you, so you won't waste more time in the future. Second - remember that this will leave a content for the future followers. 

This is not a guide on content creating, so last advice: publish content regularly. I'm posting each Monday and Friday. It keeps me motivated and pushes me to stay outside my comfort zone to learn new things, when knowledge is later established during content creation.

# Time Tracking ⏰

One thing is universal, and I hope you understand that, if not go through _Learning Process_ (https://academy.hackthebox.eu/module/15). To keep on learning and improving yourself you have to keep practicing and making mistakes, so you can learn from them. In case you are totally out of the juice to tackle some boxes or poke at vulnerables - go on, watch a video. Read the article. Focus on your goal and go for it, step by step. Each minute counts, for example by spending just 15 minutes each day on learning, after a month you will be richer by almost 8 hours! It doesn't sound like that much, but it keeps adding.

Find your way to keep you motivating. For example, I like monitoring stuff. Writing it down, keep the track of. So, I've tried both Harvest and Toggle to keep my work tracked. Every time that failed. Until now, when I found out the Pomodoro timer feature on Toggle.

This is statistics for today:

![Toggle Report](https://cdn.hashnode.com/res/hashnode/image/upload/v1621789645142/JFQ5zpSLD.png)

As you can see, I should focus today more on learning - but the day is not over ;)

# Set Your Goal 🏁

I set my goal: report a bug in bug bounty program and get paid. I've started controlling the amount of time I spent on learning and publishing what I've learned.

I think I'm right on track :).

> Like what you see? Join the [Hashnode.com](/join) now. Things that are awesome:

>✔ Automatic GitHub Backup

>✔ Write in Markdown

>✔ Free domain mapping

>✔ CDN hosted images

>✔ Free in-built newsletter service

> By using my link you can help me unlock the ambasador role, which cost you nothing and gives me some additional features to support my content creation mojo.



