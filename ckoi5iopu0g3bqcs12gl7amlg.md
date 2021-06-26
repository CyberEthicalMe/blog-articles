## Cyber Apocalypse 2021: Passphrase

# Introduction

You found one of their space suits forgotten in a room. You wear it, but before you go away, a guard stops you and asks some questions

> Complete write up for the Passphrase challenge at Cyber Apocalypse 2021 CTF hosted by HackTheBox.eu. This article is a part of a [CTF: Cyber Apocalypse 2021](https://blog.cyberethical.me/series/ctf-cyber-apocalypse-2021) series. You can fork all my writeups directly from the [GitHub](https://github.com/KamilPacanek/writeups).

> Learn more from additional readings found at the end of the article. I would be thankful if you mention me when using parts of this article in your work. Enjoy!

***
# Contents
1. [Introduction](#introduction)
2. [Basic Information](#basic-information)
3. [Target of Evaluation](#target-of-evaluation)
4. [Analysis](#analysis)
5. [Debugging](#debugging)
6. [Flag](#flag)
7. [Additional readings](#additional-readings)
***

# Basic Information

|  # |  |
|:--    |:--|
|Type    |CTF / Reversing
|Name    | **Cyber Apocalypse 2021 / Passphrase**
|Started | 2021/04/23
|URLs    | https://ctf.hackthebox.eu/ctf/82 
| | https://ctftime.org/event/1304
|Author	| **Asentinn** / OkabeRintaro
|		| [https://ctftime.org/team/152207](https://ctftime.org/team/152207)

%%[bmac-button]

# Target of Evaluation 

We are given the `passphrase` file.

# Analysis

First things first, analyze what we have with the `file` tool:

```
$ file passphrase 

passphrase: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked,
interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=60f6b6064d2e34a2b6a24dda9feb943b0b8c360f, not stripped
```

**Symbols are left within the executable**. This is good as it makes working with the file easier.

Running `strings`. Between the output lines I've got the following:

```
Halt! 
You do not look familiar..
Tell me the secret passphrase: 
[31m
Intruder alert! 
[32m
Sorry for suspecting you, please transfer this important message to the chief: CHTB{%s}
;*3$"
```

It looks like it is going to do some string comparisons maybe..

Check for dynamically linked libraries:

```
$ ldd passphrase 

	linux-vdso.so.1 (0x00007ffc0d5b6000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ff3f9750000)
	/lib64/ld-linux-x86-64.so.2 (0x00007ff3f9b32000)
```

Nothing out stands, just regular libraries. Also, none is missing.

Symbols are included in the executable (_not stripped_) so lets list them:

```
$ nm passphrase                                      

0000000000202010 B __bss_start
0000000000202028 b completed.7698
                 w __cxa_finalize@@GLIBC_2.2.5
0000000000202000 D __data_start
0000000000202000 W data_start
0000000000000890 t deregister_tm_clones
0000000000000920 t __do_global_dtors_aux
0000000000201d78 d __do_global_dtors_aux_fini_array_entry
0000000000202008 D __dso_handle
0000000000201d80 d _DYNAMIC
0000000000202010 D _edata
0000000000202030 B _end
                 U fgets@@GLIBC_2.2.5
0000000000000bb4 T _fini
0000000000000960 t frame_dummy
0000000000201d70 d __frame_dummy_init_array_entry
0000000000000dfc r __FRAME_END__
0000000000201f70 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
0000000000000c94 r __GNU_EH_FRAME_HDR
0000000000000780 T _init
0000000000201d78 d __init_array_end
0000000000201d70 d __init_array_start
0000000000000bc0 R _IO_stdin_used
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
0000000000000bb0 T __libc_csu_fini
0000000000000b40 T __libc_csu_init
                 U __libc_start_main@@GLIBC_2.2.5
00000000000009c6 T main
                 U printf@@GLIBC_2.2.5
000000000000096a T printstr
                 U putchar@@GLIBC_2.2.5
                 U puts@@GLIBC_2.2.5
00000000000008d0 t register_tm_clones
                 U setbuf@@GLIBC_2.2.5
                 U sleep@@GLIBC_2.2.5
                 U __stack_chk_fail@@GLIBC_2.4
0000000000000860 T _start
0000000000202020 B stdin@@GLIBC_2.2.5
0000000000202010 B stdout@@GLIBC_2.2.5
                 U strcmp@@GLIBC_2.2.5
                 U strlen@@GLIBC_2.2.5
0000000000202010 D __TMC_END__
                 U usleep@@GLIBC_2.2.5
```

We can see that at the address 0x00202010 it does string compare. So it confirms the assumption from before.

Dump Procedure Linkage Table:

```
$ objdump -j .plt -S passphrase 

passphrase:     file format elf64-x86-64


Disassembly of section .plt:

00000000000007a0 <.plt>:
 7a0:	ff 35 d2 17 20 00    	pushq  0x2017d2(%rip)        # 201f78 <_GLOBAL_OFFSET_TABLE_+0x8>
 7a6:	ff 25 d4 17 20 00    	jmpq   *0x2017d4(%rip)        # 201f80 <_GLOBAL_OFFSET_TABLE_+0x10>
 7ac:	0f 1f 40 00          	nopl   0x0(%rax)

00000000000007b0 <putchar@plt>:
 7b0:	ff 25 d2 17 20 00    	jmpq   *0x2017d2(%rip)        # 201f88 <putchar@GLIBC_2.2.5>
 7b6:	68 00 00 00 00       	pushq  $0x0
 7bb:	e9 e0 ff ff ff       	jmpq   7a0 <.plt>

00000000000007c0 <puts@plt>:
 7c0:	ff 25 ca 17 20 00    	jmpq   *0x2017ca(%rip)        # 201f90 <puts@GLIBC_2.2.5>
 7c6:	68 01 00 00 00       	pushq  $0x1
 7cb:	e9 d0 ff ff ff       	jmpq   7a0 <.plt>

00000000000007d0 <strlen@plt>:
 7d0:	ff 25 c2 17 20 00    	jmpq   *0x2017c2(%rip)        # 201f98 <strlen@GLIBC_2.2.5>
 7d6:	68 02 00 00 00       	pushq  $0x2
 7db:	e9 c0 ff ff ff       	jmpq   7a0 <.plt>

00000000000007e0 <__stack_chk_fail@plt>:
 7e0:	ff 25 ba 17 20 00    	jmpq   *0x2017ba(%rip)        # 201fa0 <__stack_chk_fail@GLIBC_2.4>
 7e6:	68 03 00 00 00       	pushq  $0x3
 7eb:	e9 b0 ff ff ff       	jmpq   7a0 <.plt>

00000000000007f0 <setbuf@plt>:
 7f0:	ff 25 b2 17 20 00    	jmpq   *0x2017b2(%rip)        # 201fa8 <setbuf@GLIBC_2.2.5>
 7f6:	68 04 00 00 00       	pushq  $0x4
 7fb:	e9 a0 ff ff ff       	jmpq   7a0 <.plt>

0000000000000800 <printf@plt>:
 800:	ff 25 aa 17 20 00    	jmpq   *0x2017aa(%rip)        # 201fb0 <printf@GLIBC_2.2.5>
 806:	68 05 00 00 00       	pushq  $0x5
 80b:	e9 90 ff ff ff       	jmpq   7a0 <.plt>

0000000000000810 <fgets@plt>:
 810:	ff 25 a2 17 20 00    	jmpq   *0x2017a2(%rip)        # 201fb8 <fgets@GLIBC_2.2.5>
 816:	68 06 00 00 00       	pushq  $0x6
 81b:	e9 80 ff ff ff       	jmpq   7a0 <.plt>

0000000000000820 <strcmp@plt>:
 820:	ff 25 9a 17 20 00    	jmpq   *0x20179a(%rip)        # 201fc0 <strcmp@GLIBC_2.2.5>
 826:	68 07 00 00 00       	pushq  $0x7
 82b:	e9 70 ff ff ff       	jmpq   7a0 <.plt>

0000000000000830 <sleep@plt>:
 830:	ff 25 92 17 20 00    	jmpq   *0x201792(%rip)        # 201fc8 <sleep@GLIBC_2.2.5>
 836:	68 08 00 00 00       	pushq  $0x8
 83b:	e9 60 ff ff ff       	jmpq   7a0 <.plt>

0000000000000840 <usleep@plt>:
 840:	ff 25 8a 17 20 00    	jmpq   *0x20178a(%rip)        # 201fd0 <usleep@GLIBC_2.2.5>
 846:	68 09 00 00 00       	pushq  $0x9
 84b:	e9 50 ff ff ff       	jmpq   7a0 <.plt>

```

Nothing to see here.

[Back to top](#contents) ⤴

# Debugging

> 🔔 `CyberEthical.Me` is maintained purely from your donations - if you would like to boost the community, consider one-time sponsoring at the 🍻 [Buymeacoffee](https://www.buymeacoffee.com/asentinn) or use the [Sponsor](https://blog.cyberethical.me/sponsor) button.

Now that we know what to look for, let's try `strace ./passphrase` and `ltrace ./passphrase`.

`strace` didn't show what we look for (and we look for string comparison R-value)

```
clock_nanosleep(CLOCK_REALTIME, 0, {tv_sec=0, tv_nsec=30000000}, NULL) = 0
clock_nanosleep(CLOCK_REALTIME, 0, {tv_sec=1, tv_nsec=0}, 0x7fffeba20570) = 0
fstat(0, {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0x2), ...}) = 0
brk(NULL)                               = 0x559459cdd000
brk(0x559459cfe000)                     = 0x559459cfe000
read(0, fdfdf
"fdfdf\n", 1024)                = 6
write(1, "\33[31m", 5)                  = 5
write(1, "\n", 1
)                       = 1
clock_nanosleep(CLOCK_REALTIME, 0, {tv_sec=0, tv_nsec=30000000}, NULL) = 0
```

But `ltrace` - voilà:

```
strlen("\nTell me the secret passphrase: "...)                   = 32
sleep(1)                                                         = 0
fgets(dsada
"dsada\n", 40, 0x7f8568531980)                             = 0x7ffc986e2da0
strlen("dsada\n")                                                = 6
strcmp("3xtr4t3rR3stR14L5_VS_hum4n5", "dsada")                   = -49
printf("\033[31m")                                               = 5
strlen("\nIntruder alert! \360\237\232\250\n")                   = 22
```

So after presenting a secret passphrase, the executable gives us the flag:

```
strlen("\nTell me the secret passphrase: "...)                   = 32
sleep(1)                                                         = 0
fgets(3xtr4t3rR3stR14L5_VS_hum4n5
"3xtr4t3rR3stR14L5_VS_hum4n5\n", 40, 0x7f461a61c980)       = 0x7ffd79ab8ec0
strlen("3xtr4t3rR3stR14L5_VS_hum4n5\n")                          = 28
strcmp("3xtr4t3rR3stR14L5_VS_hum4n5", "3xtr4t3rR3stR14L5_VS_hum4n5") = 0
puts("\342\234\224"✔
)                                             = 4
printf("\033[32m")                                               = 5
printf("\nSorry for suspecting you, pleas"..., "3xtr4t3rR3stR14L5_VS_hum4n5"
Sorry for suspecting you, please transfer this important message to the chief: CHTB{3xtr4t3rR3stR14L5_VS_hum4n5}
```

[Back to top](#contents) ⤴

# Flag

**CHTB{3xtr4t3rR3stR14L5_VS_hum4n5}**

> Like what you see? Join the [Hashnode.com](/join) now. Things that are awesome:

>✔ Automatic GitHub Backup

>✔ Write in Markdown

>✔ Free domain mapping

>✔ CDN hosted images

>✔ Free in-built newsletter service

> By using my link you can help me unlock the ambasador role, which cost you nothing and gives me some additional features to support my content creation mojo.

[Back to top](#contents) ⤴

# Additional readings

> 📌 Follow the `#CyberEthical` hashtag on the social media

> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)

> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)

> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)

* [How to Analyze an ELF Executable File](https://workinjapan.today/hightech/linux-lessons-how-to-analyze-elf-files/)
* [man ltrace](https://man7.org/linux/man-pages/man1/ltrace.1.html)
* [man strace](https://man7.org/linux/man-pages/man1/strace.1.html)
* [man ldd](https://man7.org/linux/man-pages/man1/ldd.1.html)
* [man nm](https://man7.org/linux/man-pages/man1/nm.1p.html)
* [man objdump](https://linux.die.net/man/1/objdump)

[Back to top](#contents) ⤴

*Cover photo by [Ales Nesetril](https://unsplash.com/@alesnesetril?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*