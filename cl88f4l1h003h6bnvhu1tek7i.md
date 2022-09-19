## How to dual boot encrypted Kali Linux with Windows 11

%%[join-cta]

# Prerequisites

* Windows 11 already installed and running
* Free space on drive where we are going to install the Kali Linux
* Bootable USB drive with Kali Linux

# Preparation

Follow the Kali installation as usual until you encounter the partitioning step - choose "Manual partitioning". This screen is the main one we are going to come back to after each of partitioning operation. I'll be using the highlighted free space to setup the Linux partitions.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663398953282/UaCBIoqB-.png align="center")

# Create non-encrypted `boot` partition

1. Double click the free space (or click *Continue* with *Free space* selected).
* Select *New partition*. Set it to 1.8GB.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663399528463/8Sx5hJZSx.png align="left")
* Create at the beginning of free space.
* Set the values as shown below.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663451826970/L9oPhLnfu.png align="left")
* Confirm setting up partition.
* Rest of the free space assign `ext4` - it will be the filesystem encrypted partition.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663442992438/hEby-VNl4.png align="left")

> ❗ When writing a guide, I've tried different solutions for placing the boot partition. Now I know it should be set as ext4, /boot mount point, not bootable and not encrypted. I've tried to change the images to correct ones, but I could omit a few. Please just remember ext4, not bootable.

# Prepare encrypted filesystem partition

1. Select *Configure encrypted volumes*
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663450180046/YNoArUTuJ.png align="left")
* Please notice that this step cannot be undone. After you confirm, boot and filesystem (which we are going to encrypt) will be formatted. Select *Yes* and *Continue*.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663401368100/rQvjr5ah5.png align="left")
* Confirm creating encrypted volumes
* Select the filesystem partition (which was left after we created `boot` in the first phase).
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663407163966/JjeR_wm1D.png align="left")
* Remove the `/` mount point (select it and choose *Do not mount*) and confirm default encryption settings.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663408234093/GlNm1kQ4h.png align="left")
* Select *Finish*
* Confirm erasing the partition before encryption.
* Type the encryption password.
* After process finished you should end up with the similar setup.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663451975939/He-yCXY-R.png align="left")
* Choose *Configure the Local Volume Manager*. Confirm.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663416569538/_hMNZID6q.png align="left")

# Configuring LVM
You are welcomed with the following screen.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663417659528/pYk6y4nP5.png align="center")

1. Create new **volume group** "kali-vg1" and use the whole filesystem partition.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663419032944/erIq1xBXy.png align="left")
* Create **logical volumes** called `swap`, `root`, `var`, `tmp` and `home`.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663419835609/C6gcv7nSh.png align="left")
> ℹ Only `swap` and `root` are required. I'm using separate `var` and `tmp` mainly [because of security reasons](https://blog.cyberethical.me/how-to-install-kali-on-the-virtual-machine#heading-choose-partitioning-method), separate `home` allow me to mount that in other distribution.
* This is how it looks on my example.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663420545624/oht3a4zj-.png align="left")
> ℹ Please take these volume sizes with a grain of salt, I'm using small volumes to save on space when writing this guide (it doesn't matter for showcasing this idea).
* Select *Finish* and *Continue*.

%%[support-cta]

# Configuring mount points.


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663452051571/WE3mRTzpk.png align="center")

Now it is time to final step: assigning mount points to freshly created logical volumes.

1. Highlight the `swap` volume and click *Continue* (or double click).
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663421308641/uRgdnMO14.png align="left")
* Select *Use as* and choose *swap*. Confirm.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663421365503/fRyqqMI3g.png align="left")
* Highlight the `root` volume and click *Continue*. Fill details as shown below.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663421660126/r4S0yldRt.png align="left")
* Do the same for other logical volumes.
* This is how it looks afterwards:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663421927025/cZv5tNNh_.png align="left")
* Select "Finish partitioning and write changes to disk"
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663422844157/55Az3hZJd.png align="left")
* Confirm. Continue with the installation.

# After reboot

With the setup I provided there is one more step - optional - without it, your machine will boot up automatically to Windows. You can still boot the Kali by interrupting the booting process (F2 on my Acer) and selecting the `kali` from boot manager.

> ℹ If you want to be more *stealthy* with your Kali, so it is a bit harder to determine you are using it, it is perfectly fine to stop here.

Launch Kali either by
* interrupting the booting process and selecting Kali from boot manager,
* or restarting Windows with <kdb>Shift</kdb> key pressed when choosing *Restart* option from power menu; then select proper device to boot up Kali

Open console and type

```sh
sudo su
nano /etc/default/grub
```

Add the following line:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663449674607/45vi6Gxcc.png align="left")

Save the file and run `update-grub` command
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663449771872/FOckD4IVe.png align="left")

That's it! Now you can select both Windows and Kali on launch. You successfully achieved dual boot with Windows 11 and Kali on encrypted volume!
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663449881179/YpZnDgvx4.png align="left")

# Additional readings
%%[follow-cta]

* [How to install Kali on the Virtual Machine](https://blog.cyberethical.me/how-to-install-kali-on-the-virtual-machine)
* [How to Dual-Boot Ubuntu 20.04 (or 22.04) and Windows 10 with Encryption](https://www.mikekasberg.com/blog/2020/04/08/dual-boot-ubuntu-and-windows-with-encryption.html)
* [Kali Linux Dual Boot Setup | Encrypted Kali Linux Installation, The Logical Volume Manager](https://www.youtube.com/watch?v=bWUcb73Id9w)
* [How to Fix: Windows Not Showing in Grub Boot Menu | Kali Linux 2022 ✓](https://www.youtube.com/watch?v=REZS35S3L_Y)
