---
title: "Driver's Shadow: Unmasking the Kernel Intruder"
seoTitle: "Driver's Shadow: Unmasking the Kernel Intruder (HackTheBox 2025)"
seoDescription: "Explore the nuances of detecting and analysing rootkits in Linux memory dumps, uncovering hidden payloads and stealth hooks via forensic techniques."
datePublished: Tue May 27 2025 13:00:21 GMT+0000 (Coordinated Universal Time)
cuid: cmb6ixrq3001x09ju15gl5vuy
slug: htb-business-2025-blackout-drivers-shadow
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1748301869679/5ad9d1aa-936c-48aa-88fd-91e81c1e2c6b.png
tags: ctf, hackthebox, htb, volatility, ctf-writeup, rootkits

---

> A critical Linux server began behaving erratically under suspected Volnaya interference, so a full memory snapshot was captured for analysis. Stealthy components embedded in the dump are altering system behavior and hiding their tracks; your task is to sift through the snapshot to uncover these concealed elements and extract any hidden payloads.

**Type**: Forensics  
**Difficulty**: Hard  
**Authors**: [c4n0pus](https://www.hackthebox.com/blog/author/Odysseus%20\(c4n0pus\))  
**Event**: [Global Cyber Skills Benchmark CTF 2025: Operation Blackout](https://www.hackthebox.com/events/global-cyber-skills-benchmark-2025) ([**ctftime**](https://ctftime.org/event/2707))

Official write-up:  
TBA (*but probably* [*here*](https://github.com/hackthebox/business-ctf-2025))

# Lessons learned/Topics:

* volatility Linux symbols generation
    
* ftrace hooks
    
* rootkit concealment and persistence
    

# Initial recon

%%[support-cta] 

Given: `mem.elf`

```plaintext
$ mem.elf (2.170.445.516 bytes)
ELF 64-bit LSB core file, x86-64, version 1 (SYSV)
62760d9d1ae0cc321c0ffa6a41fbfea30e280e51c38643eee0aee371d64ea12d  mem.elf
```

As usual with any kind of memory dumps, the tool of choice is [Volatility Framework](https://github.com/volatilityfoundation/volatility3).

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">The way I work with <code>volatility</code> is via the Python virtual environment, so before I run any commands, I’m activating it <code>. /opt/volatility3/venv/bin/activate</code>. That way I can simply call <code>volatility</code> instead of <code>python3 /opt/volatility/vol.py</code></div>
</div>

Getting to know with memory dump of what image I’ll be working on:

```plaintext
$ vol -f ../mem.elf banners.Banners                                                       
Linux version 6.1.0-34-amd64 (debian-kernel@lists.debian.org)
(gcc-12 (Debian 12.2.0-14+deb12u1) 12.2.0, GNU ld (GNU Binutils for Debian) 2.40) 
#1 SMP PREEMPT_DYNAMIC Debian 6.1.135-1 (2025-04-25)
```

This is quite a recent kernel, and we haven’t been given a symbols file - it could mean that we may have to prepare a symbols file ourselves. Only one way to verify the hypothesis:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748288571388/058b6c58-c095-4cf3-bff1-d05e36b1395b.png align="center")

No symbols, no output. Let’s obtain them then.

# Generating Intermediate Symbols File

<details data-node-type="hn-details-summary"><summary>ISF (Intermediate Symbols File)</summary><div data-type="detailsContent">Structured representation of a program's debug symbol information, typically extracted from compiled binaries. It captures details about data types, structures, function prototypes, and memory layouts in a format that's easier to process or analyse. IFS files are often used in reverse engineering, debugging, or binary analysis workflows to understand how a program operates internally without relying on source code.</div></details>

ISF allows Volatility to

* Understand kernel data structures like tasks, modules, file descriptors, etc.
    
* Match the memory layout from the dump to the correct kernel version and configuration.
    
* Work more efficiently, since the raw debug symbols don’t need to be parsed every time.
    

First, we have to get hands on the Linux distribution that the memory was dumped from.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748288713820/f45e76a2-56ce-41e8-8b7d-49ba9da8eb02.png align="center")

I’m downloading Debian from the official site, installing it in the virtual machine. Now we have to match the same kernel.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748288943944/56ef6c45-5515-4db2-addf-d7a047e46793.png align="center")

```plaintext
$ sudo apt install linux-image-6.1.0-34-amd64
$ sudo shutdown -r now
```

After reboot, we can verify that we have the correct version of the system.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748289370739/6ec953a0-9a81-48a7-bdec-0acbfa6f599e.png align="center")

With that, we can proceed with `dwarf2json` tool (run on the same system).

```plaintext
$ cd /opt
$ git clone https://github.com/volatilityfoundation/dwarf2json.git
$ sudo apt install golang
$ go build -buildvcs=false
$ cd ~
$ sudo /opt/dwarf2json/dwarf2json linux --elf /usr/lib/debug/boot/vmlinux-6.1.0-34-amd64 \
--system-map /boot/System.map-6.1.0-34-amd64 > Debian12-6.1.0-34-amd64.json
```

Copy the JSON file to the system with `volatility` under `volatility3/volatility3/symbols`.

# Harvesting runtime artifacts

[Challenge](https://blog.cyberethical.me/htb-cyber-apocalypse-forensics-oblique-final) after challenge, I keep running the same plugins, so I’ve created a useful script that helps me automate these first steps.

```python
import subprocess, os, sys

def main():
    if len(sys.argv) != 2:
        print(f"Usage: {sys.argv[0]} <memory_image>")
        sys.exit(1)

    memory_image = sys.argv[1]
    if not os.path.exists(memory_image):
        print(f"Error: Memory image '{memory_image}' not found.")
        sys.exit(1)

    # Configuration
    venv_activate = "/opt/volatility3/venv/bin/activate"
    output_dir = "./vol"
    os.makedirs(output_dir, exist_ok=True)

    plugins = [
        "linux.bash.Bash",
        "linux.elfs.Elfs",
        "linux.envars.Envars",
        "linux.ip.Link",
        "linux.ip.Addr",
        "linux.lsof.Lsof",
        "linux.malfind.Malfind", # never found anything useful yet, but sounds useful
        "linux.pagecache.Files",
        "linux.psaux.PsAux",
        "linux.pslist.PsList",
        "linux.psscan.PsScan", # potential hidden processes that are not listed in pslist
        "linux.pstree.PsTree",
        "linux.sockstat.Sockstat",
        "linux.lsmod.Lsmod"
    ]

    # Construct the bash command to run all plugins
    joined_plugins = " ".join(plugins)
    bash_command = f'''
    source "{venv_activate}" && \\
    for plugin in {joined_plugins}; do
        echo "[*] Running $plugin"
        vol -f "{memory_image}" $plugin > "{output_dir}/$(basename $plugin).out"
    done
    deactivate
    '''

    # Execute the bash command
    subprocess.run(bash_command, shell=True, executable='/bin/bash')

if __name__ == "__main__":
    main()
```

At this point, I’m browsing and grepping through the output files to find some suspicious entries. But to keep this article tidy, let’s just go through the questions (in no particular order).

# Questions (some)

## What is the name of the backdoor udev Rule?

<details data-node-type="hn-details-summary"><summary>udev rules</summary><div data-type="detailsContent">A <strong>udev rule</strong> is a configuration directive used by the <strong>udev device manager</strong> on Linux systems to dynamically manage device nodes in the <code>/dev</code> directory. These rules define how the system should handle hardware events, such as assigning device names, setting permissions, or triggering scripts when devices are added or removed. udev rules are typically stored in <code>/etc/udev/rules.d/</code> and are written using specific match and action syntax.</div></details>

In the output of `linux.pagecache.Files` we can see references to many `/usr/lib/udev/rules.d/` and among them there is one that is definitely not a standard: `99-volnaya.rules`. Its existence may potentially mean that the rootkit tries to integrate into the system’s normal device-handling workflow, making its actions blend in with legitimate operations.

**Answer**: `99-volnaya.rules`

## What is the resolved IP of the attacker?

When browsing through the `linux.sockstat.Sockstat` output, a couple of connections stand out:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748291985174/e0413852-263b-4235-98f9-c09fca141044.png align="center")

`bash` raised TCP connections should always give a warning.

**Answer**: `16.171.55.6`

## What is the name of the kernel module?

Because we already know that rootkit uses the `volnaya` string we can just search for its occurrences in all `volatility` outputs.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748292461588/e4bca38d-2058-445d-8d1e-a92e0f86a452.png align="center")

**Answer**: `volnaya_xb127`

## There is one bash process that is hidden in \_\_**USERSPACE\_\_**, what is its PID?

<details data-node-type="hn-details-summary"><summary>userspace</summary><div data-type="detailsContent">Refers to the part of the operating system where regular applications run, as opposed to <strong>kernelspace</strong>, where the OS kernel and its modules operate.</div></details>

There are two indicators of the suspicious `bash` process present in the `volatility` outputs.

1. In the aforementioned `linux.sockstat.Sockstat` potential revshells connections originate from one specific `bash` process.
    
2. In the `linux.psscan` output, we can identify multiple `bash` processes, but only one is a direct child of `systemd` **and** spawns an `id` command.
    

Answer: `2957`

## What is the address of \_\_x64\_sys\_kill, \_\_x64\_sys\_getdents64 (ex: kill:getdents64)?

## What **SYSCALLS** are hooked with ftrace, sorted via SYSCALL number (ex: read:write:open)?

These two will be answered by outputs of two more `volatility` outputs.

```python
volatility -f mem.elf linux.tracing.ftrace.CheckFtrace > vol/linux.tracing.ftrace.CheckFtrace.out
volatility -f mem.elf linux.check_syscall.Check_syscall > vol/linux.check_syscall.Check_syscall.out
```

**ftrace** is a kernel-level tracing framework used for debugging and performance profiling. But attackers (or rootkits) can abuse ftrace to hook syscall handlers without modifying the syscall table — making detection harder. So, for example rootkit can intercept a kill signal and perform own code in place of it (can still execute the original syscall making its action very difficult to detect for user). Or can hook to directory listing event and hide its own files before they are printed via `ls`. *Ops, spoilers*.

In the output of `linux.check_syscall.Check_syscall`:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748295182952/cbb58db5-f55a-4022-afcc-d345cf1b3031.png align="center")

Addresses of `kill` and `getdents64`: `0xffffb88b6bf0:0xffffb8b7c770` (answer).

And now the output of the second plugin:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748295345402/707f3d3d-8bc2-41ac-9203-0b3e7b5b897e.png align="center")

These are all hooks created by the rootkit. Only two of them are syscalls, though.

Answer: `kill:getdents64`

*Yes, one of the flags was included as an format hint to other question. I wonder how many players got that flag just by typing it because it was shown as example? New CTF solving routing discovered?*

# Extracting rootkit loader

For the rest of the questions, we have to extract the binaries for both `volnaya_usr` (rootkit loader) and `volnaya_xb127` (rootkit kernel module). As long as the binary is cached in the memory, we should be able to dump it with `volatility`. From the `linux.pagecache.Files`:

```plaintext
SuperblockAddr    MountPoint  Device   InodeNum   InodeAddr        FileType   InodePages   CachedPages   FileMode     AccessTime                       ModificationTime                 ChangeTime                       FilePath               InodeSize
0x9a097453c800    /           8:1      913940     0x9a0974b68128   REG        107          107           -rwxr-xr-x   2025-05-12 19:20:29.156000 UTC   2025-05-12 19:17:34.102003 UTC   2025-05-12 19:20:12.731680 UTC   /usr/bin/volnaya_usr   437032
```

We are lucky and `volnaya_usr` is cached on 107 pages with inode address `0x9a0974b68128`. Unfortunately `volnaya_xb127` is not (0 pages, and 0 inode size) - but we will take care of it soon. For now, dump the `volnaya_usr`.

```plaintext
vol -f mem.elf linux.pagecache.InodePages --inode 0x9a0974b68128 --dump
```

Open it in Ghidra.

## What is the XOR key used (ex: 0011aabb)?

In the `main` function, there is a reference to `xor_key`. When navigated to the `.data` section select the label of `xor_key` so the whole memory segment is selected and choose “Copy special..” → “Byte String (No Spaces)".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748297530042/40fca59f-e811-4499-94d7-e1ee185ede73.png align="center")

Answer: `881ba50d42a430791ca2d9ce0630f5c9`

## What is the hostname the rootkit connects to?

In the same line of `main` we can see how the hostname is created. We can navigate to the `.data` section to copy the content of the `hostname` and decode the hostname rootkit connects to.

```py
hostname = bytes.fromhex(
    "eb7ac96120c5531232c1b7ad3408c4f8a66dca612cc5491832caadac"
)

xor_key = bytes.fromhex(
    "881ba50d42a430791ca2d9ce0630f5c9"
)

decoded = bytearray()
for i in range(len(hostname)):
    decoded.append(hostname[i] ^ xor_key[i % len(xor_key)])

print(decoded.decode('utf-8', errors='replace'))
```

Answer: `callback.cnc2811.volnaya.htb`

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Bonus: in the <code>init_revshell</code> there is call<code>kill(0x41,local_c)</code>. It is usually used to send a signal to the process. But the thing is the maximum signal number is 64 and here 65 is sent.. Which doesn’t make sense until you recall the rootkit hooks to the kill syscall. This is one of the methods of communication between rootkit modules.</div>
</div>

# Extracting rootkit kernel module

Now the last two questions require looking into the previously unavailable `volnaya_xb127`. By analysing the code of the loader, especially `install_module()` function, we can see the following:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748298170445/9eeed9de-0311-4777-b675-f9f3ebf377c7.png align="center")

`syscall` is used to call `sys_init_module` (see `linux.check_syscall.Check_syscall`, index `0xAF=175`) on some binary data located under `local_18`, of size `0×666c0`, passing some argument. We can see that `local_18` is a pointer to a `deobf()` function.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748298394142/c6dd3318-c7a4-4417-84f2-dc76012f7d68.png align="center")

That’s definitely a rootkit kernel module binary. Here is the Python version (copy the `elf_body` and `elf_hdr` content from the `.data` sections.

```python
elf_hdr = bytes.fromhex("7f4..00") # 64-byte ELF header
elf_body = bytes.fromhex("8c..c9") # ELF body

xor_key = bytes.fromhex("881ba50d42a430791ca2d9ce0630f5c9")

# Decrypt body
decrypted_body = bytearray()
for i in range(len(elf_body)):
    decrypted_body.append(elf_body[i] ^ xor_key[i % 16])

# Reconstruct full ELF
with open("volnaya_xb127", "wb") as f:
    f.write(elf_hdr)
    f.write(decrypted_body)
```

Open `volnaya_xb127` in Ghidra.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748298684415/f46aa40a-13e3-42cf-9645-5a20a3bd5994.png align="center")

Here we have a nice listing of all hooks rootkit is setting and functions used to hide its activity.

## What string must be contained in a file in order to be hidden?

In each of `filldir` hooks (that rootkit uses to manipulate directory listing) we can find a filename check. If it contains `MAGIC_WORD` the file is hidden.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748298890366/1013f115-6336-4676-a667-1c04ce6b3cf3.png align="center")

Answer: `volnaya`

## What owner UID and GID membership will make the file hidden (UID:GID)?

Now our focus is moved to the `hook_sys_getdents64()` function.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748299543269/7b830470-b068-49ab-ac88-0b92ee81d20d.png align="center")

By analysing the `vfs_fstatat` [documentation](https://docs.huihoo.com/doxygen/linux/kernel/3.7/fs_2stat_8c.html#a31fb6a6cc4130b5895fa52cca267bd8f), we can determine that `lVar10` contains a structure file metadata structure. Then in UID and GID of the file are stored respectively in `iVar8` and `iVar2` (through pointer offsetting). Later, those values are compared to the `USER_HIDE` and `GROUP_HIDE` which means those last two constants are UID and GID we are looking for.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748300089486/6c5c0310-5831-4a01-ba86-de6f7398b1dd.png align="center")

By converting hexadecimals to decimals, we have the answer.

Answer: `1821:1992`

# Additional Readings

%%[follow-cta] 

* [Write-up: Forensics: Oblique Final (HTB CA 2024)](https://blog.cyberethical.me/htb-cyber-apocalypse-forensics-oblique-final)
    
* [Linux signals](https://www.chromium.org/chromium-os/developer-library/reference/linux-constants/signals/)
    
* [Build Custom Linux Profile for Volatility](https://www.iblue.team/memory-forensics-1/volatility-plugins/build-custom-linux-profile-for-volatility)