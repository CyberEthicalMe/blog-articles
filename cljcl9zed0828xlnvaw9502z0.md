---
title: "Solving a CTF using OpenAI models"
seoTitle: "Solving Securing Midsummer Corp 2023 CTF with OpenAI"
seoDescription: "Write-up for challenges from Midsummer Corp Hack CTF. AI assisted approach with BingAI and ChatGPT as security consultants."
datePublished: Mon Jun 26 2023 08:20:41 GMT+0000 (Coordinated Universal Time)
cuid: cljcl9zed0828xlnvaw9502z0
slug: solving-securing-ctf-with-open-ai
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1687692591336/c768077b-5cbb-4817-9531-4b6e7bc84141.png
tags: hacking, openai, cybersecurity-1, write-up

---

%%[follow-cta] 

# The Plan

There is something I wanted to try since I've watched videos of people creating games without any programming knowledge whatsoever. After [I have created a honeypot using the ChatGPT v3](https://blog.cyberethical.me/ive-asked-chatgpt-to-write-a-honeypot), I'm ready to solve the CTF using AI only: ChatGPT and Bing AI (mostly strict mode) - latter because it has internet access.

For the readability reason, I'm not going to paste whole conversations, but the important bits.

# Recon

For me, the competition starts before launching the first challenge. In the introductory part, I can read that

> In this room, every task will allow you to gain access to a new Midsummer Corp employee. On every account you can also find a piece of the final puzzle `fernflower_flag[1-6].png`, which you will need to complete the last quest.

It indicates that the final flag or answer (the [Crown Jewels](https://www.securityforum.org/solutions-and-insights/protecting-the-crown-jewels/)) is distributed in six parts and placed somewhere on the file systems (presumably) of six accounts. The number of challenges (Puck, Leshy, Baba Yaga, Boruta, Twardowski and Popiel) is also six - so that is a match - one account, one challenge - one part of the final flag.

It is also worth noting that two years ago [I have participated](https://blog.cyberethical.me/securing-ctf-rozdzka) in the CTF organized by the [Securing](https://www.securing.pl/) and my write-up won that year's competition. If you haven't read it yet, I strongly recommend it because it was my first experience of Securing team potential - just see what platform did they use to host the event 😉.

## The Setting

> *Legend has it that the fern flower appears on the eve of the summer solstice at the stroke of midnight. It can only be found deep in the forest, where it grows in a secret and hidden spot known only to the bravest and most skilled of seekers. Those who are lucky enough to find it and pick it up at the right moment are granted great powers and blessings and may even have their wishes come true.*

Securing chosen **Kupala Night** as a topic of this CTF.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686163668606/1bc0aed8-a213-4fd8-b558-d8b95d1101f1.png align="center")

I like that because the elusive fern flower is a great analogy to the flag that participants are looking for in the competition.

> Also, what ChatGPT didn't mention - fern flower does not exist. Ferns do not bloom.

If you are not familiar with that legend of blooming fern, here are some of my recommendations to familiarize with:

* [absolutely amazing](https://www.last.fm/user/Asentinn/library/artists?from=2023-05-08&to=2023-06-07) Polish folk rock band - [*Żywiołak*](https://open.spotify.com/artist/3EYuaOf6w8uTG7eM8vpMLH?si=ZTG_7ugcT96i41seTHy8_A) (*Elemental*); their songs focus on the Kupala Night and Slavic mythology
    
* "Kwiat Paproci" ("Fern Flower") [book series](https://lubimyczytac.pl/cykl/9374/kwiat-paproci) by [Katarzyna Berenika Miszczuk](https://lubimyczytac.pl/autor/19159/katarzyna-berenika-miszczuk)
    

Anyway, let's look at the challenges names in the context of that all - are those names chosen arbitrary or do they have some meaning that could help to solve tasks?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686169570276/c94273a1-0c1d-4bfb-b6f2-5c318f3512f7.png align="center")

And maybe bot can drop us some ideas of what the challenges may be about?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686169940018/b1b157b6-4b6a-47d4-a3e9-d5b7a04d46e1.png align="center")

Interesting - but here is crucial to remember one thing - when OpenAI model doesn't know the answer, it comes up with some ([here](https://community.openai.com/t/gpt-4-keeps-lying-instead-of-saying-i-dont-know/218549/2), [here](https://community.openai.com/t/qa-fine-tuned-chatbot-not-answering-from-the-trained-data-but-nonfactual/21999/21?page=2) and [here](https://ai.stackexchange.com/a/38272)). This is just how these kinds of model works. For example, I've asked a Bing AI to find some information about one of the challenges suggested by ChatGPT - and it looks like the bot just made that one up.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686169834011/cece0cf7-72d0-4823-973b-2ed2ba3aea42.png align="center")

## The Platform

> The application is based on the NextCloud server ([GitHub - nextcloud/server](https://github.com/nextcloud/server)). The software and the configuration have been intentionally made vulnerable.

According to the GitHub it is a PHP server with JavaScript frontend for the data storage/management.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686170887074/ab519bd5-01d9-4047-abeb-6f35ac036027.png align="center")

Tag cloud:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686170871783/a9af6b63-3720-4a4f-b2fb-719c77cb1281.png align="center")

Interestingly, there is a `hacktoberfest` tag there. I've dug deeper and found out Nextcloud is participating in [Hacktoberfests](https://hacktoberfest.com/) in the years 2016 and 2017 ([post from 2016](https://nextcloud.com/blog/nextcloud-android-client-1-4-0-has-been-released/)). So just for future reference, I'm adding a [pull requests list](https://github.com/nextcloud/android/pulls?q=is%3Apr+is%3Aclosed+label%3Ahacktoberfest) applied during that event because of the following reasons:

* Securing team could be using an older version of Nextcloud Server,
    
* Securing team could be introducing/reverting some pull requests to include vulnerabilities,
    
* both
    

Ok, let's start with the actual challenge.

# Midsummer Corp (sanity check)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686206345293/50385eba-20c2-4e00-a50f-51879ad176f7.png align="center")

> For the sake of consistency - most actions I'll be performing on the Kali Linux, partially becasue I couldn't connect `openvpn` on Windows.

I'm connecting to the VPN `sudo openvpn thm-eu1.vpn` and test the connection using `curl -IL 10.10.21.207` to roughly see what headers are being exchanged. Some of them:

```plaintext
Server: Apache/2.4.56 (Debian)
X-Powered-By: PHP/8.1.18

X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'none';base-uri 'none';manifest-src 'self';script-src 'self';style-src 'self' 'unsafe-inline';img-src 'self' data: blob:;font-src 'self' data:;connect-src 'self';media-src 'self';frame-ancestors 'self';form-action 'self'
```

#### Bing AI on HTTP Headers

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686206082394/55dade19-af9a-40b6-a5ad-0a02693be096.png align="left")

The sanity check question is to find the base URL of the application. When you look closely - on the footer there is some text (which often contains redirection to the landing page). This is how the page looks like when adding a following CSS rule.

```css
* {
  color: white;
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686206558330/1d32d32e-3647-4692-aaf6-21a6cfd4e200.png align="center")

So I hover on the **Midsummer Corp** link - it leads to `https://files.midsummer.corp.local/`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686206734698/28516042-d7c6-490a-8e72-8092b6ccdf40.png align="center")

I'm adding that to the hosts and browsing the page again. Unfortunately, the page seems not to be served over HTTPS.

```plaintext
# THM
10.10.21.207 files.midsummer.corp.local
```

**From now on, I'll be using the Request Blocking feature of Firefox to not load the background image** - the page looks much easier to browse and the size of the image is huge!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686207816934/2f182f95-c3b3-4b57-847e-2552b23a2ec5.png align="center")

## Source Code

To be honest - that's the longest `.htaccess` file I've ever seen - 132 lines.

```plaintext
$ wc -l .htaccess
132 .htaccess
```

I've then asked Bing AI to explain to me what some of the sections of that file are doing.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686208408743/627ed370-fa6b-4933-9876-eea6929865c1.png align="center")

When comparing this file with the current version on the GitHub - there are additional lines under the `#### DO NOT CHANGE ANYTHING ABOVE THIS LINE ####` comment.

```plaintext
ErrorDocument 403 /index.php/error/403
ErrorDocument 404 /index.php/error/404
<IfModule mod_rewrite.c>
  Options -MultiViews
  RewriteRule ^core/js/oc.js$ index.php [PT,E=PATH_INFO:$1]
  RewriteRule ^core/preview.png$ index.php [PT,E=PATH_INFO:$1]
  RewriteCond %{REQUEST_FILENAME} !\.(css|js|svg|gif|png|html|ttf|woff2?|ico|jpg|jpeg|map|webm|mp4|mp3|ogg|wav|wasm|tflite)$
  RewriteCond %{REQUEST_FILENAME} !/core/ajax/update\.php
  RewriteCond %{REQUEST_FILENAME} !/core/img/(favicon\.ico|manifest\.json)$
  RewriteCond %{REQUEST_FILENAME} !/(cron|public|remote|status)\.php
  RewriteCond %{REQUEST_FILENAME} !/ocs/v(1|2)\.php
  RewriteCond %{REQUEST_FILENAME} !/robots\.txt
  RewriteCond %{REQUEST_FILENAME} !/(ocm-provider|ocs-provider|updater)/
  RewriteCond %{REQUEST_URI} !^/\.well-known/(acme-challenge|pki-validation)/.*
  RewriteCond %{REQUEST_FILENAME} !/richdocumentscode(_arm64)?/proxy.php$
  RewriteRule . index.php [PT,E=PATH_INFO:$1]
  RewriteBase /
  <IfModule mod_env.c>
    SetEnv front_controller_active true
    <IfModule mod_dir.c>
      DirectorySlash off
    </IfModule>
  </IfModule>
</IfModule>
```

### Version 26.0.0 (`/status.php`)

Requesting `status.php`

![status.php](https://cdn.hashnode.com/res/hashnode/image/upload/v1686209668409/234b75e9-5f15-4755-be8f-3e8000b5378b.png align="center")

Ok so, unfortunately, it's not as outdated as I thought it would be - the current version at the time of writing this article is `26.0.2`. Couple of interesting commits **missing** in the `26.0.0`

🔸 [sec(deps): Update guzzlehttp/psr7](https://github.com/nextcloud/server/commit/da95d3389d1d25039f45a4a01099153fccd08117)

Updates guzzlehttp/psr7 from version `2.4.3` to `2.4.5` - here is [the diff](https://github.com/nextcloud/3rdparty/compare/0ec73636ee36558960a1df60828b645ef3d4a53c...a7f0f5d93a0d9a947b4d3676d6344824d8490eab). Basically HTTP headers validation.

🔸 [fix(security): Mark recording\_servers key appconfig as private as it contains a secret](https://github.com/nextcloud/server/commit/da95d3389d1d25039f45a4a01099153fccd08117)

Well, that could be worth checking because of some "secret".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686211598580/b693dee1-a8c8-46a7-8f39-4a77ab61860a.png align="center")

🔸 [clear encrypted flag when moving away from encrypted storage](https://github.com/nextcloud/server/commit/40748731f130fd79d609bf80016fefe30f9b06c8)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686211748394/eff7a8d5-51cd-4d33-8196-0319d9e8a541.png align="center")

🔸 [hide shared files located in group folder's trash bin](https://github.com/nextcloud/server/commit/6b4644ba1a85f4ff36bdefb3fac99dfd32d70f9b)

Maybe some data exfiltration from the shared files by looking in the group folder's trash bin?

🔸 [fix(files\_sharing): Allow file actions other than download for hide download shares](https://github.com/nextcloud/server/commit/b7aad07df571c2a972ab589cfcd4ed10440b3e1c)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686212156383/f8e724e3-14ab-4a31-a605-4414be8475d2.png align="center")

Maybe the file download action is still available?

🔸 [**fix(lostpassword): Also rate limit the setPassword endpoint**](https://github.com/nextcloud/server/commit/e4b987142c9b1a562460ee1ed3cc75613a3d5a9e)

Paired with [feat(security): Allow to opt-out of ratelimit protection, e.g. for testing on CI](https://github.com/nextcloud/server/commit/96204fe6e986785c9de2a4d3809d62709e576ffa) by the same author. Adding some comments/attributes on `core/Controller/LostController->setPassword(..)` and response throttling - most certainly to prevent brute-forcing.

🔸 [fix: Always create user directory when transferring files to new users](https://github.com/nextcloud/server/commit/bf564185d111960dc33b3665339f59086df7aac2)

Another way to unauthorized read?

### `/cron.php`

Requesting `cron.php`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686213690574/043c046d-a969-494f-aea3-daa30e29639b.png align="center")

### `/ocs/v1.php`, `/ocs/v2.php`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686213820921/23fc7eec-74d1-4687-9831-72392496e4a4.png align="center")

Ok, I'm deciding to not chase the rabbit and start doing the challenges.

%%[support-cta] 

# Puck (intern)

> *Responsible for creating chaos and confusion to distract people from the company's true intentions*

It does sound mischievous.

From the task description, we know that the Puck's password is accessible for us, somewhere in the application - and that the brute-force attack is not required. To be honest, I'm surprised that I haven't stumbled upon the plaintext password so far, as it is often "hidden" in the page source, some page or known URL (like `robots.txt`).

So I'm going back to the page source of the "Direct log in". I see there is some quite informative JS code in the `head` tag

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686214731029/b1a5d473-3d49-4ae5-a880-f51de603f843.png align="center")

I am pasting it in the VSCode, and I'm seeing something useful.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686214787187/8b511f2d-26f5-4952-9d1d-e244bd62c8ff.png align="center")

And success - first credentials are `puck:sfLfSNYavTD4PL4Z`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686214954530/ce8d01ac-e814-48d6-8c89-0450bb4c65c4.png align="center")

Getting the flag from `Fern_flower_ritual_shard1.txt`. I'm also downloading a part of the final flag (`fernflower_flag1.png`).

Now, I noticed that 1 file is hidden (information under the file list). So, I went to *Files settings* and enabled *Show hidden files*.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686225530693/a8c9d5ce-a7cd-47fb-a219-2f968ab62d83.png align="center")

> On the sidenote, `research` folder contains `*.txt` files with fern AI-generated content (verified on three AI detectors) :). And some screens from social media
> 
> ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686226549706/c9d6bf9e-fb64-4cf0-96e1-62f8c81b9835.png align="center")
> 
> Profile: [https://www.facebook.com/fernflowers17/](https://www.facebook.com/fernflowers17/), original content:
> 
> ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686226876714/7e60b500-6eb5-4f9f-9ee6-6a9ce89f61ce.png align="center")

In the hidden folder `.mail` I found mails coming from different members of the fern team (like Baba Yaga and Boruta). One of the emails is crucial for me

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686227210600/38cccf1c-c6e5-4bcc-9269-4bd849e46bcb.png align="center")

That helped me answer the last question in Puck challenge and get Leshy credentials (`leshy:nQRbhRyxuDuU9GNd`).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686227276832/86bda05c-8d0e-478a-8a40-a5ef13d5d99b.png align="center")

# Leshy (**Forest Tracker)**

> *Leshy is a forest spirit who has an intimate knowledge of the forest and its inhabitants. His job is to lead the team to the locations where the fern flower grows and to keep a watchful eye on any unwanted visitors who may threaten their mission*

It is worth noting that we are given a link to [OWASP resource](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html) that could be helpful to bypass MFA on a Leshy account.

I started by browsing the source code, to see the MFA flow - and at first, I thought that I had found that the token is constant (`234567`) - but it didn't work.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686228623324/dbf1266c-6191-4184-bd8b-b29fcff6d6e0.png align="center")

So, I have to consult my assistants :)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686240911881/51484c04-af43-4326-a7ce-27a222d122e6.png align="center")

And now I'm asking ChatGPT directly on that topic, knowing that the `234567` is crucial to solving that task.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686241097922/12925b51-acf9-4f2a-b599-df5a29e1601a.png align="center")

Ok, now I've asked it to give me the example in Python.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686242207878/f2acdda7-3b7f-40e8-b3cb-e52afe071dfe.png align="center")

After a few retries I have finished with that:

```python
#!/usr/bin/python
import pyotp
import base64

def generate_totp_code(seed_value):
    totp = pyotp.TOTP(seed_value)
    return totp.now()

seed_value = base64.b32encode("234567".encode())
totp_code = generate_totp_code(seed_value)
print(totp_code)
```

Let's try it. Unfortunately, it doesn't work.

After unveiling the hint - of course, I can use a different user to add MFA to it, so I can have the code generator (because seed is the same, OTP also are the same).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686234307655/f5ac493a-1f3d-4fb8-8d98-91e888a7933b.png align="center")

> Backup codes are not the same.

Because that secret code is the same for each user, we can use the same authentication codes generated for the `puck`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686234524993/45d96809-0875-49fc-992b-6601c75212d0.png align="center")

I'm downloading both the flag and final flag PNG.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686234624535/4823749e-b59f-4b36-b98c-941d4030de37.png align="center")

No additional findings are reported. Because it is a dedicated instance - I'm removing MFA from both users, in case I have to log in as one again.

# Baba Yaga (**Witchcraft Researcher)**

> *Baba Yaga is an expert in dark magic and is responsible for researching and discovering new ways to harness the power of the fern flower for the company's purposes. Her extensive knowledge of spells and potions makes her an invaluable member of the team.*

The task here is to come up with a rate limit bypass and brute-force the password reset token - possible by using HTTP headers modifications.

> Remember when we listed some changes that were introduced in `26.0.2` but missing in `26.0.0`? This sounds closely related to the scope of the task.

First, let's ask ChatGPT about those comments on `LostController.php`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686244609477/2d463f6e-4865-4d0f-b1c2-59f66e4383f7.png align="center")

## `/Middleware/Security/RateLimitingMiddleware.php`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686244571863/8e892ec0-1194-4c4b-9c2a-33782d36cff9.png align="center")

And in the same Middleware:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686244825826/ba7242e1-d3f4-455d-861b-96dea41f172d.png align="center")

That should be easy. In `routes.php` there is an endpoint for that password reset:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686245954600/b8876ab2-f946-400e-9c23-4344543309e3.png align="center")

So, I fire up Burpsuite and proxy browser, intercept the request too `/lostpassword/reset/form/{token}/babayaga` and then add `X-Forwarded-For` header.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686246299631/d0366df3-afa1-4da4-aed8-5625947075c5.png align="center")

I am setting a new password for `babayaga` - that request also has to be intercepted and enhanced with `X-Forwarded-For`. After the password is changed, I log in as `babayaga` and extract the flag and shard.

> Endpoint for resetting a password. I left it over for the next day becasue I couldn't find the exact value that is expected to be entered on the submission form. In the `routes.php` there is a `/lostpassword/reset/form/{token}/{userId}` but this is not an acceptable answer. The correct one is `/lostpassword/reset/form/<TOKEN>/<USER>` which could be found in the `LostController->email(..)`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686291340954/9ed6f59f-3c85-470e-8214-424266a6c14b.png align="center")

No additional steps are to be done on that account.

# **Boruta (Trickster)**

> *Boruta is a mischievous and unpredictable figure, known for his ability to shape-shift and his love of pranks and practical jokes. As the company's Trickster, Boruta uses his skills to distract and confuse their enemies, often leading them astray with illusions and false leads. Despite his playful nature, Boruta is fiercely loyal to the company and will stop at nothing to help them achieve their goals.*

Boruta's account can be accessed only with "app password". Maybe it is related to the WebDAV interface I've been seeing on each account.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686292740091/e0d11993-fc95-40a3-90dd-51353ec73cdd.png align="center")

I can map the `leshy` files

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686294344323/3bab26fa-4629-43b7-984e-294b5a2cb247.png align="center")

But I can't do that obviously with `boruta` because I don't know his password - if he has set any.

So perhaps WebAuthn?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686292829102/9995f13d-6cc3-40f3-a934-351d527b6902.png align="center")

That also didn't seem to work. I'm reading the description once again. And looking at the Settings -&gt; Security page.

* "Prerequisites\*\*:\*\* Access to any account."
    
* "Can you figure out a way to use an app password?"
    
* "Mass Assignment vulnerability"
    

And finally, it clicked when I saw there is an input I haven't seen before

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686296905747/cb54ee9e-3bc6-458b-a6c8-b206f9ff9e2a.png align="center")

Ok, so probably it is possible to cheat the API and add an app password to `boruta` instead of the currently logged-in user.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686297289511/96c1547d-baa4-488a-9cb2-2b090f987bdf.png align="center")

Ok, let's see the source code for that.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686298050473/bd8b4ddb-cff5-414f-9174-836db35058c1.png align="center")

It looks like it is a right area - because of the allowed users collection with `boruta` name in it.

When I add `loginName` parameter

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686297841634/fed70d67-5727-4040-b313-2013ce364084.png align="center")

Makes sense because the first check means exactly that - if `loginName` is `boruta` and it is not empty - throw that firewall error. So to bypass it and simultaneously skip the second check - I just have to add whitespaces to the `boruta`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686298031535/92a48aa9-f75d-4ed7-95ba-f5839c725d91.png align="center")

Now, I'm using the method of mounting files via DAV interface

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686302508474/9ff0b903-a520-4a10-99ae-970878c45b23.png align="center")

> Remember to use explicitly `-o ro`, without it you won't be able to even read the contents of the file.
> 
> ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686301957603/39875570-457a-49cb-9473-70a52e26d1d8.png align="center")

I'm copying the files I'm interested in.

```plaintext
sudo cp /mnt/dav-boruta/*.txt /eth/securing-midsummer-corp-2023
sudo cp /mnt/dav-boruta/fernflower_flag5.png /eth/securing-midsummer-corp-2023
```

One of the files contains a flag.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686302893815/509978c4-2617-4c85-89ab-843b9093fbed.png align="center")

On Boruta's account, there are additional files related to CTF:

* `Fern_flower_residual_shard4.txt` - looks like a more AI-generated story
    
* `fernflower_flag5.png` - I don't know if that's an error, but so far, we have 1-3 and 5 final flag parts (Boruta is 4th account)
    
* `twardowski_sso_config.msg.txt` - message from Twardowski to Boruta about SSO configuration
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686304242196/bf8aebfd-75db-4fdb-b842-d5daaf6aa62d.png align="center")
    
* `twardowski_sso_mess_msg.txt` - just information to Boruta about the mentioned SSO config
    

I have tried to use that endpoint, asked ChatGPT to generate me some sample of SAML Response - but to no avail. Let's go to the next task.

# **Twardowski (Alchemist) - unsolved**

> *Twardowski is a master of alchemy and uses his knowledge to create potions and elixirs that aid in the search for the fern flower. He is constantly experimenting with different ingredients and formulas to unlock the secrets of the flower's power. Twardowski's work is crucial to the company's mission, as his potions can provide the team with enhanced strength, speed, and intelligence.*

Getting some help.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686314449513/5944369c-7ed4-44b1-9c5e-67f7262e86ee.png align="center")

I've dug deeper in the application, looking for some SAML references. Because when looking at how the SAML Assertion is made, it felt like some metadata for that response is needed or at least useful.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686322522280/da5470e3-ae8b-49b8-8abd-ae0f738e7025.png align="center")

I have updated the ChatGPT with the metadata XML, then proceeded to look for the other APIs in `custom_apps\user_saml\appinfo\routes.php`. I noticed `apps/user_saml/saml/login` endpoint and I viewed it via browser. I got the following redirection:

`http://idp.midsummer.corp:5000/saml?SAMLRequest=nZJPbxoxEMXvlfIdor2z613CQixAoqF%2FkCggIDn0Eg32bGLJa7seb5t%2B%2B5hl29JIySFznJn383sjjwlq7fisCY9miz8apHDx4TLWU60N8XY6SRpvuAVSxA3USDwIvpt9W%2FIiZdx5G6ywOnmpe1sGROiDsqbTLeaTZL36tFx%2FWazupRQjkcMBqxxQ9IUEHJajCisYlgPByqsCrkaHPnTaO%2FQUSZMkgmOrAxI1uDAUwIQ4YUW%2Fx8oeu97nJWcjXhTfO%2FU8ZlYGQkt4DMHxLFPSpbWS1NQ1%2BlRY7%2FiAMZYdc3WyTZf7ozJSmYe30x5OS8S%2F7veb3ma923eU2Z8z3FgTX0O%2FQ%2F9TCbzdLv%2BaqZRGemEn1VaAzsA5ypqIuD86a%2B1lICiZnujjY4O3l%2FDT99FqDCAhwDg7Z53hHV%2FFuIv5xmolfp8Gx%2FpsfQ3h9bPkad52lOxV7SpvDDkUqlIok3%2Bcmdb2141HCDhJgm8wuczi%2Byc%2F%2F%2F%2Fc6TM%3D&RelayState=http%3A%2F%2Ffiles.midsummer.corp.local%2Fapps%2Fuser_saml%2Fsaml%2Flogin`

I've asked my assistant if I can generate a SAML Assertion having the `SAMLRequest`.

> I must admit that I'm following blindly the AI here, becasue I've never worked with SAML Auth.

A couple of messages later, I had a plan - and I was even given a Python lines to decode the SAMLRequest.

```python
import base64
import zlib

saml_request = "nZJPbxoxEMXvlfIdor2z613CQixAoqF/kCggIDn0Eg32bGLJa7seb5t++5hl29JIySFznJn383sjjwlq7fisCY9miz8apHDx4TLWU60N8XY6SRpvuAVSxA3USDwIvpt9W/IiZdx5G6ywOnmpe1sGROiDsqbTLeaTZL36tFx/WazupRQjkcMBqxxQ9IUEHJajCisYlgPByqsCrkaHPnTaO/QUSZMkgmOrAxI1uDAUwIQ4YUW/x8oeu97nJWcjXhTfO/U8ZlYGQkt4DMHxLFPSpbWS1NQ1+lRY7/iAMZYdc3WyTZf7ozJSmYe30x5OS8S/7veb3ma923eU2Z8z3FgTX0O/Q/9TCbzdLv+aqZRGemEn1VaAzsA5ypqIuD86a+1lICiZnujjY4O3l/DT99FqDCAhwDg7Z53hHV/FuIv5xmolfp8Gx/psfQ3h9bPkad52lOxV7SpvDDkUqlIok3+cmdb2141HCDhJgm8wuczi+yc////c6TM="
decoded_data = base64.b64decode(saml_request)
decompressed_data = zlib.decompress(decoded_data, -zlib.MAX_WBITS)
```

```xml
<samlp:AuthnRequest xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" 
  xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion"
  ID="ONELOGIN_ddc8c1abef1aec3cdae768fefa765c0642a48b3a" Version="2.0"
  IssueInstant="2023-06-09T16:08:22Z" Destination="http://idp.midsummer.corp:5000/saml"
  ProtocolBinding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
  AssertionConsumerServiceURL="http://files.midsummer.corp.local/apps/user_saml/saml/acs">
  <saml:Issuer>http://files.midsummer.corp.local/apps/user_saml/saml/metadata</saml:Issuer>
  <samlp:NameIDPolicy Format="urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified"
    AllowCreate="true" />
</samlp:AuthnRequest>
```

Later I found out that all I need is the [SAMLTool site](https://www.samltool.com/decode.php). As for the SAML response validations goes, I found that

* SAMLResponse has to be sent via POST parameter (`application/x-www-form-urlencoded`) - `Auth->processResponse()` - application is using `HTTP_POST Binding`; parameter has to be base64 encoded (`Response` constructor)
    
* cookie `saml-data` has to be set and its value is provided in the initial response to GET `http://files.midsummer.corp.local/apps/user_saml/saml/login` - `SAMLController->assertionConsumerService()`
    
* validation is done in `Response->isValid()` - this method has ~300 lines with some portions commented out
    

Some of the requirements come from the settings - which I cannot see, so it's a bit hard to guess what went wrong.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686346163306/42fc7c34-c9a5-4f06-8878-466cb673b36f.png align="center")

This is my final ResponseXML that still is not valid. I have also tried both the assertion and message signing with a certificate generated on the [SAMLTool](https://www.samltool.com/self_signed_certs.php).

```xml
<samlp:Response xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol"
    xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion"
    ID="_8e8dc5f69a98cc4c1ff3427e5ce34606fd672f91e6" Version="2.0"
    IssueInstant="2023-06-21T17:26:53Z">
    <saml:Issuer>http://idp.midsummer.corp</saml:Issuer>
    <samlp:Status>
        <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success" />
    </samlp:Status>
    <saml:Assertion xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:xs="http://www.w3.org/2001/XMLSchema" ID="_d71a3a8e9fcc45c9e9d248ef7049393fc8f04e5f75"
        Version="2.0" IssueInstant="2023-06-21T17:26:53Z">
        <saml:Issuer>http://idp.midsummer.corp</saml:Issuer>
        <saml:Subject>
            <saml:NameID SPNameQualifier="http://sp.example.com/demo1/metadata.php"
                Format="urn:oasis:names:tc:SAML:2.0:nameid-format:transient">
                _ce3d2948b4cf20146dee0a0b3dd6f69b6cf86f62d7</saml:NameID>
            <saml:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
                <saml:SubjectConfirmationData NotOnOrAfter="2024-01-18T06:21:48Z"
                    Recipient="http://files.midsummer.corp.local/apps/user_saml/saml/acs" />
            </saml:SubjectConfirmation>
        </saml:Subject>
        <saml:Conditions NotBefore="2023-06-21T17:26:53Z" NotOnOrAfter="2023-06-26T17:26:53Z">
        </saml:Conditions>
        <saml:AuthnStatement AuthnInstant="2023-06-21T17:26:53Z"
            SessionNotOnOrAfter="2023-06-26T17:26:53Z"
            SessionIndex="_be9967abd904ddcae3c0eb4189adbe3f71e327cf93">
            <saml:AuthnContext>
                <saml:AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:Password</saml:AuthnContextClassRef>
            </saml:AuthnContext>
        </saml:AuthnStatement>
        <saml:AttributeStatement>
            <saml:Attribute Name="alias"
                NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:basic">
                <saml:AttributeValue xsi:type="xs:string">twardowski</saml:AttributeValue>
            </saml:Attribute>
            <saml:Attribute Name="role"
                NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:basic">
                <saml:AttributeValue xsi:type="xs:string">sso</saml:AttributeValue>
            </saml:Attribute>
        </saml:AttributeStatement>
    </saml:Assertion>
</samlp:Response>
```

Above XML is not complete - but I've seen that some of the properties like \`InResponseTo\` are not validated if not present - I have omitted them here.

# Last Resort

Because I couldn't progress further without `twardowski` account, my journey ends here. Or is it?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687677514699/a986208c-6d7c-499a-910e-2c99ca401253.png align="center")

I have 4/6 pieces and that is enough to get the part of the phrase used to create a flag.

`The fern flower reveals ...`

Because of how THM presents the answer text boxes, I also know the exact length of the flag.

```xml
**************{***********************************}
Midsummer_Corp{Th3_f3r**f!0w3r_r3**@1*************}
```

Creative Bing AI wasn't much of a help.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687687698530/b0cdbb8b-ca17-44e8-9c7d-41edbbbbb7f5.png align="center")

ChatGPT responses make me wonder why AIs have issues counting the length of the word.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687690327622/9e1da19a-3e27-40c7-a3df-9ea9880ba9f1.png align="center")

Brute forcing answers is against the rules.. So I can only guess it has something to do with the saying "*The fern flower reveals itself only to the bravest*".

# Conclusions

%%[join-cta] 

* Would it be possible to solve the whole CTF using **only** an AI processor? Despite the fact, it was a bit of pain to use in this event - I really think it shines when it comes to pure binary and mathematical operations. I've seen reverse challenges solved after one prompt, but it should be understandable that it won't happen for tasks that require pentesting and live analysis of the application flow.
    
* It takes longer to prompt (guide) the AI through the challenge, but when you are in total black - I think it is easier and more efficient to do. This is especially true with Bing AI which links you more resources to read on the topic which can give you additional insight.
    

Overall, I had fun solving the challenges, and definitely I want to see the solution for that SAML task - to see what did I did wrong.