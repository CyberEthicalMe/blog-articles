# How to reinstall Linux on encrypted LVM

> Please note that following approach works only when Linux was installed previously with a separate `/home` partition

> This guide assumes you know how to install the Linux already. If not, please refer to [previous article](https://blog.cyberethical.me/dual-boot-windows-11-encrypted-kali).

> I'm using the Kali installer but steps should be same regardless the Linux flavour

# Background

Story starts when I tried to get my CUDA working with a `hashcat`. I have installed video card drivers and tools with `apt get install nvidia-driver nvidia-cuda-tools` which leads to some serious issues that prevent me from logging into my installation.

I haven't found the complete guide on how to reinstall the system, leaving the `/home` (or simply any other) partition intact. So, whoever may want to do the same, here is what I did.

# Unlock the LVM

%%[join-cta]

1. Boot the system with Live CD Kali installation.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1667132249519/tv3tmRn1-.png align="left")
> Verify the checksum of downloaded file with the one presented on the download page
* Boot into Rescue mode.
![kali-rescue.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1667134911031/uSAVpIDPR.gif align="left")
* Follow through the installation the usual way.
> Can't confirm if you have to choose the same values as it was on original installation, but it shouldn't matter. Let me know in the comments if that causes some issues on you
* On *Detecting disks* step, installer should detect the encrypted volume and ask you for the passphrase. Enter it.

# Partitioning

1. When asked to choose root do not use any, instead click *Go Back* to display the installation steps.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1667137705554/45UrUJeBV.png align="left")
* Skip the *Enter rescue mode* and select the *Partition disks step*.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1667137602307/ETcp3rI7Q.png align="left")
* Choose the manual partitioning. Now configure mounting points as it was done before. To not lose the `/home` data, leave the volumes size as it is. Choose to format every partition, **but do not format **`home`.
![kali-lvm-root.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1667138196118/7CG4bV16w.gif align="left")
* Mount home partition as `/home`, make double sure you are not formatting it.
![kali-lvm-home.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1667139581069/2tHEoACDd.gif align="left")
* Select boot partition, choose `Ext4` and `/boot` mount point.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1667140410358/Gs1EAfXCY.png align="left")
> You can keep the existing data, because the encrypted volume itself does not change, only the LVM it contains.

This is how the partitioning schema looks like before changes are committed.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1667140797778/CCTpIjQY_.png align="left")
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1667140889231/DRLuRhGoe.png align="left")
Now continue with the installation.

# Update `crypttable`

%%[support-cta]

After the booting on fresh system, Grub won't recognize/discover the encrypted volume, and it fails booting. It can show that device is not available or simply show `initramfs` prompt. Reason is that installer is not updating the `/etc/crypttab` file - probably because during the partitioning step, we actually didn't configure the encrypted partition; it was unlocked already before we entered the partitioning screen. To fix that, perform the following steps. *Original author of this is [Halacs from superuser.com](https://superuser.com/a/1595535/1730908)*.

1. Boot the Live CD again, but this time actually start the Live System.
> At the time of writing default credentials are `kali/kali`.
* Open terminal. Enter root shell by typing `sudo su`.
* Find out what is the volume path for `/boot` and encrypted LVM. You will recognize file system by its size.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1667152618184/2wKFA_PNl.png align="left")
* Unlock the encrypted volume.
```sh
$ cryptsetup luksOpen /dev/sda6 gabor2-crypt
```
> If you care of naming your encrypted volume (it will be visible on passphrase prompt each boot) change the `gabor2-crypt` to your liking and use that name consistently during further steps.
* After decryption, LVM content should be visible in both `fdisk -l` and `lsblk`
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1667154271639/bRsryTQeS.png align="left")
* Use `/mnt` to mount all new system and boot partitions. You will be switching root to the `/mnt` later, so it is important to keep all mount points in the single folder.
```sh
mount /dev/mapper/kali--vg-root /mnt
mount /dev/mapper/kali--vg-home /mnt/home
mount /dev/mapper/kali--vg-var /mnt/var
mount /dev/mapper/kali--vg-tmp /mnt/tmp
mount /dev/sda5 /mnt/boot
```
* Verify that mount points are correct using `lsblk`
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1667155208774/Ync0Fm3bW.png align="left")
* Now mount other critical directories, without this further commands will fail.
```sh
mount --bind /dev /mnt/dev
mount --bind /run /mnt/run
mount --bind /proc /mnt/proc
mount --bind /sys /mnt/sys
```
* Switch the root to `/mnt`.
```sh
chroot /mnt
```
*From now on, until leaving the `chroot`ed scope, all commands are performed on the newly installed system*
* Get the UUID of the encrypted volume to write to `crypttab`.
```sh
blkid /dev/sda6
```
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1667165395929/xdRbZq72h.png align="left")
* Place these in the `crypttab`.
```sh
$ nano /etc/crypttab
gabor2-crypt UUID="f8da3b2d-c0d4-4d02-b45d-9917adf31931" none luks
```
> Please note that this is not copy-paste from previous step.
* Update initramfs and grub.
```sh
update-initramfs -u
update-grub
```
> Watch out for warning messages, when device label in `/etc/crypttab` does not match current label of decoded LUKS. If that happen, please ensure the name in `/etc/crypttab` and name of encrypted volume are the same.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1667167582123/cJj2EI5Tb.png align="left")
* Now exit `chroot` scope, unmount all devices and close LUKS.
```
exit
umount -R /mnt
vgchange -a n
cryptsetup luksClose gabor2-crypt
```
* Restart.

Now your system should show passphrase prompt and boot to login screen. Unfortunately, you can't login to your user because it does not exist yet on the new system.

If you are using the system that doesn't block `root` account password login, you can skip the next section.

# Boot into `root` shell

1. Boot the system. Stop on GRUB loader. Highlight the "Kali" entry.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1667168628132/pZmOf6Umb.png align="left")
* Press `E`. Search for the line starting with `linux`.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1667169267732/UgHmOzc6I.png align="left")
* Change the ending to `rw quiet init=/bin/bash`. Press F10.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1667338714902/Ib2sfQ_GT.png align="left")
* Unlock the LUKS.

# Add user back

1. Add user, add to sudoers.
```sh
useradd --no-create-home -d /home/asentinn -s /bin/bash asentinn 
chown -R asentinn:asentinn /home/asentinn
sudo -l -U asentinn
passwd asentinn
```
2. Reboot. Now you can login as sudo user with your old home partition.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1667339468994/QIDJ4giUW.png align="left")

# Additional readings

%%[follow-cta]

* [How to dual boot encrypted Kali Linux with Windows 11](https://blog.cyberethical.me/dual-boot-windows-11-encrypted-kali)
* [Superuser: How can I reinstall Ubuntu focal 20.04 on an existing LUKS encrypted system?](https://superuser.com/a/1595535/1730908)
* [Superuser: No mouse/keyboard input on Linux](https://superuser.com/questions/1745835/no-mouse-keyboard-input-on-linux-update-nouveau)
* [chroot(1) - Linux man page](https://linux.die.net/man/1/chroot)
