## Android malware analysis: preparation


> 
This is a first article from **Malware Analysis** series where I'm presenting how you can safely grab the malicious code and by using various techniques like decompiling and deobfuscation see what it is doing. In these series, you are going to see different approaches and solutions that are used to bypass antivirus and other threat defenses, learn how to approach payloads and hopefully recognize common patterns that are reused by malicious actors. Enjoy!

***
# Contents
1. [Download ISO](#download-iso)
2. [Create Virtual Machine](#create-virtual-machine)
3. [Install Android system](#install-android-system)
4. [Network](#network)
5. [Snapshots](#snapshots)
6. [Additional readings](#additional-readings)
***

> 🔔 `CyberEthical.Me` is maintained purely from your donations - consider one-time sponsoring with the [Sponsor](/sponsor) button or 🎁 [become a Patron](https://www.patreon.com/cyberethicalme) which also gives you some bonus perks. 
Join our [Discord Server](https://discord.com/invite/5MjU4Cxf3R)!

# Download ISO

> In this guide I'm using the Virtual Box, but you can use any other virtualization software.

 Go to the [official Android x86 project site](https://www.android-x86.org/download.html) - download page will list a few mirrors from which you can choose from. After navigating to the mirror site, Grab the Android ISO version you are interested in. 
 
 > I'm downloading a 9.0 R2 version from [Open Source Development Network](https://osdn.net/projects/android-x86/releases).

![2021-09-18-12-35-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632067842337/BydMHQfFv.png)

When download finished, verify the SHA-256 hash.

> See: *[Why MD5 and SHA-1 are considered no longer trustworthy](https://security.stackexchange.com/questions/87375/why-are-md5-and-sha-1-still-used-for-checksums-and-certificates-if-they-are-call)*.

> See [Verifying hash of the downloaded file](https://blog.cyberethical.me/how-to-install-kali-on-a-raspberry-pi#download-image)

[Back to top](#contents) ⤴

# Create Virtual Machine

Open Virtual Box and click `New`.

![](assets/2021-09-18-17-06-52.png)
![2021-09-18-17-06-52.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632067867159/usoOjvzWF.png)

Choose `Linux 2.6/3.x/4.x (32-bit)` or `Linux 2.6/3.x/4.x (64-bit)`.

![2021-09-18-17-08-10.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632067978411/EWYAvwI9f.png)

Set 4GB RAM.

![2021-09-19-14-35-30.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632067992796/qyhoi4Hx2.png)

Leave the default `Create a virtual hard drive now`.

![2021-09-18-17-12-19.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068002453/PUfZWiSsI.png)

Leave VDI and dynamic allocation. Assign 8GB (default) size.

![2021-09-18-17-13-01.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068008553/crawZ9dSk.png)
![2021-09-18-17-15-36.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068013544/h89GRviT7.png)

> If possible save the disk file on the SSD. I've encountered significant slowliness when I initialy have it on the HDD.

# Install Android system

Run the virtual machine and mount the ISO you have previously downloaded.

![2021-09-18-17-20-48.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068024206/jkB_TLgEC.png)

Click `Start`. Choose installation option. Select `Create/Modify partitions` by pressing <kbd>C</kbd>.

![2021-09-18-17-31-30.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068036372/v3ffrcAYe.png)

[Decline](https://askubuntu.com/questions/944936/grub-wont-boot-after-converting-mbr-partition-table-to-gpt) GUID Partition Table usage.

![2021-09-18-17-32-27.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068046184/z_SzrcCVO.png)

Create a new primary partition from the entire free space.
![2021-09-18-17-59-15.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068059087/-FxVl7ecS.png)
![2021-09-18-18-00-15.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068072602/ceCO2Pfuc.png)

Mark it as a bootable and `Write`. Type `yes` confirming choices and wait until process is completed.

![2021-09-18-18-01-19.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068096697/E_2ihIIF4.png)

`Quit` - you will be back at partition selection screen.

![2021-09-18-18-05-06.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068085660/5s3xzrX4R.png)

Select the newly created partition.

![2021-09-18-18-31-06.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068110684/D_UpZeYXU.png)

Format it using `ext4` and confirm selection.
![2021-09-19-00-48-04.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068118607/cGbWxnr_3.png)

Install GRUB.

![2021-09-19-01-36-54.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068154486/Jk9p7Fxov.png)

Make `/system` directory read/write.
![2021-09-19-01-37-47.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068149608/cwNMk041v.png)

After installation is completed, it doesn't matter what you will choose because GRUB will appear either way.

![2021-09-19-02-42-35.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068144425/Kx2KT8Ltv.png)
![2021-09-19-08-10-05.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068136995/xFJBtMhP_.png)

[Back to top](#contents) ⤴

## Common issue: no GUI

When booting for the first time, you could have two issues
* system is not booting at all
* Android is booting to shell

![2021-09-19-08-38-49.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068202017/0FGN_00IS.png)

Solution for this is setting graphics controller to **VBoxVGA** (`Settings -> Display -> Screen`) and disable 3D acceleration.
![2021-09-19-08-40-54.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068195034/Sj1DV21yW.png)

> Do you like what you see? Join the [Hashnode.com](https://blog.cyberethical.me/join) now and start publishing. Things that are awesome:  
>✔ Automatic GitHub Backup  
>✔ Write in Markdown  
>✔ Free domain mapping  
>✔ CDN hosted images  
>✔ Free built-in newsletter service  
>✔ Built-in blog monetizing through the Sponsor feature  
> By using my link, you can help me unlock the ambassador role, which cost you nothing and gives me some additional features to support my content creation mojo.

[Back to top](#contents) ⤴

# Network

By default, Virtual Box configures [NAT](https://www.nakivo.com/blog/virtualbox-network-setting-guide/), so you don't have to do any additional configuration. Upon entering the system, you will be allowed to select Wi-Fi Connection called `VirtWifi`. After connection is established, you are ready to go.

![2021-09-19-13-17-41.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632068174588/yKd-DFr7B.png)

# Snapshots

Turn off the system and create a snapshot of the fresh state of the virtual machine. Do the same before any malicious software testing.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1632080722863/a1RVBpn5F.png)

[Back to top](#contents) ⤴

# Additional readings

> 📌 Follow the `#CyberEthical` hashtag on the social media  
> 🎁 Become a Patron and [gain additional benefits](https://www.patreon.com/cyberethicalme)  
> 👾 Join CyberEthical [Discord server](https://discord.com/invite/5MjU4Cxf3R)  
> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)  
> 👉 LinkedIn: [CyberEthical.Me](https://www.linkedin.com/company/cyberethical-me)  
> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)  
> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)  

* [VirtualBox - Android x86 - Don't boot in GUI but just in command line](https://superuser.com/questions/1395714/virtualbox-android-x86-dont-boot-in-gui-but-just-in-command-line)
* [VirtualBox Network Settings: Complete Guide](https://www.nakivo.com/blog/virtualbox-network-setting-guide/)
* [Grub won't boot after converting MBR partition table to GPT](https://askubuntu.com/questions/944936/grub-wont-boot-after-converting-mbr-partition-table-to-gpt)
* [Why are MD5 and SHA-1 still used for checksums and certificates if they are called broken?](https://security.stackexchange.com/questions/87375/why-are-md5-and-sha-1-still-used-for-checksums-and-certificates-if-they-are-call)
* [Android-x86 - Porting Android to x86](https://www.android-x86.org)

[Back to top](#contents) ⤴
