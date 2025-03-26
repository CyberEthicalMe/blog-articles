---
title: "Forensics: ToolPie"
seoTitle: "HackTheBox CTF - Forensics: ToolPie"
seoDescription: "Write-up for the ToolPie challenge from HackTheBox Cyber Apocalypse 2025 CTF event. Forensics category, PCAP and Python bytecode analysis."
datePublished: Wed Mar 26 2025 23:19:36 GMT+0000 (Coordinated Universal Time)
cuid: cm8qjrbii000008lddqbs20lg
slug: forensics-toolpie
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1743030958560/b229451f-fe21-4f0f-a63b-0a9864e059a4.png
tags: python, hacking, forensics, hackthebox, pcap, wireshark

---

# Introduction

> In the bustling town of Eastmarsh, Garrick Stoneforge’s workshop site once stood as a pinnacle of enchanted lock and toolmaking. But dark whispers now speak of a breach by a clandestine faction, hinting that Garrick’s prized designs may have been stolen. Scattered digital remnants cling to the compromised site, awaiting those who dare unravel them. Unmask these cunning adversaries threatening the peace of Eldoria. Investigate the incident, gather evidence, and expose Malakar as the mastermind behind this attack.

**Type**: Forensics  
**Difficulty**: Medium  
**Event**: [Hack The Box Cyber Apocalypse 2025: Tales From Eldoria](https://www.hackthebox.com/events/cyber-apocalypse-2025) ([ctftime](https://ctftime.org/event/2674/))

Official write-up by thewildspirit:  
[https://github.com/hackthebox/cyber-apocalypse-2025/tree/main/forensics/ToolPie](https://github.com/hackthebox/cyber-apocalypse-2025/tree/main/forensics/ToolPie)

%%[support-cta] 

# Initial recon

Given:

```plaintext
$ sha256sum *
03b6b7b65e4a2cd1b02b35ef814ce79c2d6746436394f7b42757cb0c4d79f88a  capture.pcap

$ wc -c *
8613560 capture.pcap
```

Archive contains only `capture.pcap` - which given its size (over 8.5 MB) it can be expected to contain some binary data exchange.

# Investigation

Start `wireshark` and let’s see what we have there.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743011438168/52dbe107-5751-4f86-bc38-37164f9d6d9a.png align="center")

Brief scrolling through the packets show a lot of TCP packets. Borrowing the presentation technique from the [author](https://github.com/hackthebox/cyber-apocalypse-2025/tree/main/forensics/ToolPie) let’s do not infer the conclusions based on the glance of such inferior and unreliable tool like an eye. **Statistics** &gt; **Protocol Hierarchy** - yes, majority of traffic is TCP data.

![Statistics > Packet Hierarchy](https://cdn.hashnode.com/res/hashnode/image/upload/v1743012469649/68713fa9-64a9-45a1-9ceb-955d7ed8952d.png align="center")

We can go furhter and check **Statistics** &gt; **Conversations:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743013176688/cc312b12-4a4f-4168-88d8-cfb6b041be91.png align="center")

One route stands out and it’s the one that transferred ~9MB of data to the `13.61.7.218` on port `55155`. Because we’ve already seen the HTTP requests, ports `:80` and `:443` should not be that suspicious (but only in this CTF scenario, usually `:80` is something that should be given special care).

# 01\. What is the IP address responsible for compromising the website?

We have one IP on the radar - but so far we only know that it is where *probably* exfiltrated data were sent. We still don’t know who initiated the attack. Because question especially asks about the website, let’s check all HTTP traffic.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743014233698/657ef9f6-80cb-4361-9698-ee068a647abb.png align="center")

One specific raw stands out - `/execute` coming from `194.59.6.66`. Packet details ultimately points that this indeed was mallicious intent.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743014428973/acac6e48-d008-4a85-b45c-3a29d3f58e32.png align="center")

Answer: `194.59.6.66`

# 02\. What is the name of the endpoint exploited by the attacker?

Already found.  
Answer: `execute`

# 03\. What is the name of the obfuscation tool used by the attacker?

Decode the payload sent to the `/execute` endpoint. Right click on `JavaScript Object Notation: application/json`, **Show Packet Bytes**, select **Show as JSON** and save. Little clean up and we have stage 1, python script:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743018354935/fcf80d84-f6e2-4d06-af00-4fa21c6e9246.png align="center")

<details data-node-type="hn-details-summary"><summary>Python Data Marshalling</summary><div data-type="detailsContent">Read and writing Python values in a binary format. The format is specific to Python, but independent of machine architecture. The marshal module exists mainly to support reading and writing the “pseudo-compiled” code for Python modules of .pyc files.</div></details>

We can further analyze the object.

```python
import marshal,lzma,gzip,bz2,binascii,zlib

compressed_data = b'BZh91AY&SY\x8d <<binary data>>'
marshalled_object = bz2.decompress(compressed_data)
code = marshal.loads(marshalled_object)

print(type(code))

# List all attributes and methods of the object
print("Available attributes and methods:", dir(code))

# Filter and show only callable methods
methods = [m for m in dir(code) if callable(getattr(code, m))]
print("Callable methods:", methods)

if hasattr(code, "co_consts"):
    print("Constants:", code.co_consts)
else:
    print("No constants found")

if hasattr(code, "co_names"):
    print("co_names:", code.co_names)
else:
    print("No co_names found")

if hasattr(code, "co_varnames"):
    print("co_varnames:", code.co_varnames)
else:
    print("No co_varnames found")
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743020372119/279c8154-4ae3-46de-a350-553f93745cad.png align="center")

At this point we can see many references to the [Py-Fuscate](https://github.com/Sl-Sanda-Ru/Py-Fuscate).

Answer: `Py-Fuscate`

# 04\. What is the IP address and port used by the malware to establish a connection with the Command and Control (C2) server?

This was already established in initial recon and doubled in the output of previous script.

Answer: `13.61.7.218:55155`

# 05\. What encryption key did the attacker use to secure the data?

In the output of previous script we see references to the `Crypto.Cipher` package and AES - so with great probability we can assume that indeed AES was used to encrypt data. This alghoritm needs two parameters to work - encryption key and IV vector. Usually encryption key is constant and known for the decryptor and IV vector is different for each ciphertext. In the easiest way, IV vector is sent in plain text together with the encrypted data.

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">We can cheat a bit here (this is what I did) and come back to the PCAP, inspect all trafic that comes <strong>to</strong> the C&amp;C and discover that two packets follow this template: <code>ec2amaz-bktvi3e\administrator&lt;SEPARATOR&gt;5UUfizsRsP7oOCAq</code>. I’ve blindly typed this as a flag and it passed.</div>
</div>

For this and the next question we first have to somehow disassembly the Python bytecode. The most straighforward is to use `dis` package.

## Dissassembling Python bytecode

%%[follow-cta] 

This can be done using `dis.dis()`. One caveat with that is the method returns `None` and prints directly to the standard output. One way to catch is to redirect call to some file (`python dismal.py > source.dis`)

```python
import marshal,lzma,gzip,bz2,binascii,zlib
import dis

# Compressed data (example)
compressed_data = b'<<bytecode>>'
marshalled_object = bz2.decompress(compressed_data)
code = marshal.loads(marshalled_object)
dis.dis(code)
```

We ends with semi-readable pseudo code that looks a lot like an assembler code.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743024399598/cab5ec4a-e2d3-40a2-b58a-1332553252d1.png align="center")

It is problematic to discuss how I’ve analyzed the pseudocode, but at this point this fragment is most important to us:9

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743025183380/d9c46707-4415-4239-8e63-dd552e62fd55.png align="center")

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text"><a target="_self" rel="noopener noreferrer nofollow" href="https://stackoverflow.com/questions/12673074/how-should-i-understand-the-output-of-dis-dis" style="pointer-events: none">&gt;This&lt;</a> is a good thread, I’ve learned from it how to “decipher” <code>dis.dis()</code> output:</div>
</div>

This can be translated to Python code:

```python
cypher = AES.new(key.encode(), AES.MODE_CBC, key.encode())
```

And here is where the attacked made a mistake - in his encryption alghoritm used the same value for both encryption key and IV vector.. and passed that value in plaintext together with the encrypted data.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743025644880/9a819878-e318-4637-91c9-c25f21c6576e.png align="center")

```python
client.send((user+SEPARATOR+k).encode()) 
```

Inspecting TCP packets sent to C&C:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743025847907/14931cd8-0695-4a65-ad6f-53236054baf4.png align="center")

Answer: `5UUfizsRsP7oOCAq`

# 06\. What is the MD5 hash of the file exfiltrated by the attacker?

We have all the information needed to recover the file.

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">For automated approach I stronly suggest reading author’s <a target="_self" rel="noopener noreferrer nofollow" href="https://github.com/hackthebox/cyber-apocalypse-2025/tree/main/forensics/ToolPie" style="pointer-events: none">write-up</a>.</div>
</div>

In `wireshark` **Analyze** \&gt; **Follow** … &gt; **TCP Stream**. Find stream with big chunks of data, select only communication to C&C, raw (important later) and save.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743026129603/acf39e75-689d-44cc-902e-8c08c4b1bf89.png align="center")

Now open up [CyberChef](https://gchq.github.io/CyberChef/). Input AES paremeters we know.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743026574558/530075d9-1fe4-44d1-9d2d-0fc8dbb587dd.png align="center")

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Padding is really important. When you choose correct padding (CBC, not “CBC no padding“), decrypted output should be clean, without any symbols. I was unable to get the correct MD5 becasue of that - PDF files was readable, had correct number of bytes.. but hash did not match.</div>
</div>

With this we can further decode all top 4 lines from the `wireshark` view.

1. Some bytes
    
2. `check-ok`
    
3. `ok`
    
4. `8504240`
    

When you hover over the packet details in the “Follow TCP Stream” you can see from which packet data comes from:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743026826786/4c173eac-a7cb-4f85-afca-76acb1d2088a.png align="center")

Clear all filters and navigate to packet no. 91 - now two packets before, in 89, there is request from C&C for which response was number `8504240`. Select packet 89, select Data segment, right click and copy value. This is another way to acquire the packet data from a single packet.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743027101787/721ebdf9-ce96-4ee2-a116-8fabd0884b3f.png align="center")

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">At this point you may realize the best is to just select <strong>Entire conversation </strong>in “Follow TCP Stream” window if you want to check all the communication, instead all of this. And you would be right.</div>
</div>

Decrypted value gives us information that C&C requested a PDF file and 8504240 is its size in bytes.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743027323918/2fd3d6fb-c0e7-4d00-9cd5-b0e46eb89095.png align="center")

Now, come back to the “Follow TCP Stream” window, ensure only traffic to C&C is selected and save. Do hexdump of the saved data.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743029099644/569d5e2c-24a0-4e6a-b830-0b2dfe066ef8.png align="center")

Notice that we have an unwated data at the beginning of the encrypted file. Content starts from `e1 4c fe a8` (see the content of packet 94 or 5th line in “Follow TCP Stream”). Now, to carve out the unncessary data, we have to remove that many bytes (`e1` starts after 69 bytes in hexadecimal base):

$$(60+9)_{(16)} = 105_{(10)}$$

```diff
$ dd skip=105 bs=4096 iflag=skip_bytes if=raw.enc of=rawdata.enc
```

The result should look like this

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743029956182/5b6597ef-b16e-4727-bc98-8bc450cf91de.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743029990663/18f83807-7e5e-4bf5-97ab-3b31b5e40806.png align="center")

Now upload file to the CyberChef and decode the PDF file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743030094647/215b0724-e27b-4b71-bdd3-470e0378d38a.png align="center")

Save the file and calculate its MD5, or just add MD5 block in CyberChef.

Answer: `8fde053c8e79cf7e03599d559f90b321`

# Bonus: pycdc

I have tried other ways to acquire something closer to the Python code - but the best what I had was running following with the Python 3.13:

```python
import marshal,lzma,gzip,bz2,binascii,zlib
import importlib

# Compressed data (example)
compressed_data = b'<<bytecode>>'

marshalled_object = bz2.decompress(compressed_data)
code = marshal.loads(marshalled_object)
pyc_data = importlib._bootstrap_external._code_to_timestamp_pyc(code)

with open("mal_313.pyc", "wb") as out_file:
    out_file.write(pyc_data)
```

Then running `pycdc` ([https://github.com/zrax/pycdc](https://github.com/zrax/pycdc)) on acquired PYC file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743024764308/26f6cc4d-0500-4066-9297-a342400a02b5.png align="center")

This was most certainly to the fact that `Py-Fuscate` was used with combination of unsupported Python versions for `pycdc` and `decompyle3`.

%%[join-cta]