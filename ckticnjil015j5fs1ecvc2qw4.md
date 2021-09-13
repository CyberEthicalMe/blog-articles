## THM: Crash Course Pen Testing

# Basic Information

| #     |   |
|:--    |:--|
|Type    | Regular Box|
|Name    | **Try Hack Me  / CC: Pen Testing**|
|URLs    | https://tryhackme.com/room/ccpentesting|
|Author  | **Asentinn** / OkabeRintaro|
|       | [https://ctftime.org/team/152207](https://ctftime.org/team/152207)|

%%[bmac-button]
***
# Contents
1. [Basic Information](#basic-information)
2. [Recon](#recon)
3. [Cracking user password](#cracking-user-password)
4. [Elevating privileges](#elevating-privileges)
5. [Additional readings](#additional-readings)
***

> 🔔 `CyberEthical.Me` is maintained purely from your donations - consider one-time sponsoring with the [Sponsor](/sponsor) button or 🎁 [become a Patron](https://www.patreon.com/cyberethicalme) which also gives you some bonus perks.  
Join our [Discord Server](https://discord.com/invite/5MjU4Cxf3R)!

# Recon

Target IP is `10.10.113.202` - I'm assigning that to the variable for ease of use.

```
$ IP=10.10.113.202
```

Scanning for open ports
```
$ nmap -sC -sV -p- $IP -oN nmap-$IP.out
```
![2021-08-31-20-40-46.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630478443722/cfQTnE0we.png)

And prepare input for the `searchsploit`

```
$ nmap -sC -sV -p 22,80 $IP -oX nmap-$IP.xml
$ searchsploit --nmap nmap-10.10.113.202.xml
```
![2021-08-31-20-45-31.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630478450512/RA4r_WnUl.png)

Firing up `nikto` and `fuff` for practice

```
$ nikto -h $IP -o nikto-$IP.txt
```
![](assets/2021-08-31-22-00-38.png)
![2021-08-31-22-00-38.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630478462352/-41AYbaGR.png)

```text
$ ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u http://$IP/FUZZ -recursion -recursion-depth 1 -e .txt,.php -v -of md -o fuzz-$IP.md
```

> `ffuf` command can be a little complicated, so let me explain it a bit  
* `-w`: wordlist for fuzzing  
* `-u`: target URL
* `-recursion`, `-recursion-depth`: when `fuff` finds a directory, it starts another scan after the current finished (you will recognize it by `Job [1/X]` label)
* `-e`: useful one, simultaneously tries to look for files with listed extensions - be careful with this one though, as it multiplies the amount of work by N where N is a number of extensions (because for each wordlist entry it tries appending these extensions).
* `-v`: shows full URL of the findings (useful when using `-recursion` flag)
* `-of`: output format, `ffuf` output files are not the easiest one to read, but and I choose the Markdown for now
* `-o`: and this is just a name for the output file; `$IP` will resolve variable name and the result 


![2021-08-31-22-00-00.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630563580588/GHv4TTHG-.png)

[Back to top](#contents) ⤴

# Cracking user password

Both find out the `/secret/` directory and `fuff` further tracked the `/secret/secret.txt`.

```text
$ curl http://10.10.200.35/secret/secret.txt
```

![2021-08-31-22-02-54.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630563589714/mNTTAuf6h.png)

Which definitely is the hash of user password. I will be using john to crack it, and it could be run blindly on that file, but lets use the `hash-identifier` that comes with Kali to see the output just out of curiosity.

```text
$ hash-identifier 046385855FC9580393853D8E81F240B66FE9A7B8
```
![2021-08-31-22-03-55.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630563602620/eWEAVhLQG.png)

As we can see it is the SHA-1 hash. Now cracking it with `john`:

```text
$ john -format=Raw-SHA1 secret.txt
```
![2021-08-31-22-06-59.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630563613057/ckZKXV02N.png)

Which was really fast (don't ever use such weak passwords, of course). So we've got credentials nyan/nyan. Try logging with these on the SSH.

```text
$ ssh nyan@$IP
```
![2021-08-31-22-07-53.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630563622218/1tNHeZKql.png)

Were in. I'm getting the user flag.

```
nyan@ubuntu:~$ cat user.txt
```

[Back to top](#contents) ⤴

# Elevating privileges

![2021-08-31-22-10-29.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630563631082/_SNrI-0II.png)

> User nyan can run /bin/su as a root without specifying its password


And just by seeing this sudoer entry we know that nyan is a can execute `sudo` command.

> Otherwise when running `sudo -l` we would see `Sorry, user nyan may not run sudo on ubuntu` (where `ubuntu` is the host name)

We got the root! So `cat` out that flag and complete the box.

```text
root@ubuntu:/home/nyan# cat /root/root.txt
```

# Additional readings

> 📌 Follow the `#CyberEthical` hashtag on the social media  
> 🎁 Become a Patron and [gain additional benefits](https://www.patreon.com/cyberethicalme)  
> 👾 Join CyberEthical [Discord server](https://discord.com/invite/5MjU4Cxf3R)  
> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)  
> 👉 LinkedIn: [Kamil Gierach-Pacanek](https://www.linkedin.com/in/kamilpacanek)  
> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)  
> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)  

* [Attacking Web Applications with Ffuf](https://academy.hackthebox.eu/course/preview/attacking-web-applications-with-ffuf)
* [GitHub: ffuf](https://github.com/ffuf/ffuf)
* [Everything you need to know about FFUF](https://codingo.io/tools/ffuf/bounty/2020/09/17/everything-you-need-to-know-about-ffuf.html)

[Back to top](#contents) ⤴
