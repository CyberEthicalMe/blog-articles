## How did I lock myself out of my server, and how did I fix it?

# Story

> Disclaimer: Following guide helps only if you have phisycal access to the server, so you can obtain the storage device. If you don't (which often happen on cloud like AWS) and you don't have backup.. well there is nothing you can do.

%%[support-cta]

Yeah, so the story is short. I was trying to restore IPv4 SSH configuration so I've changed a `sshd_config` from
```txt
Port 12345
AddressFamily inet6
#ListenAddress 0.0.0.0
#ListenAddress ::
```
to
```txt
Port 12345
AddressFamily inet4
#ListenAddress 0.0.0.0
#ListenAddress ::
```

And restarted the system (because for *some reason*, simple `sudo systemctl restart ssh` didn't work).

And after my Pi rebooted
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651343188032/UwDId88nt.png align="center")

I couldn't access my system.

For those who don't know yet, what did I wrong… Let me share something

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651338954140/i_d75FVum.png align="center")

**I typed `AddressFamily inet4` instead `AddressFamily inet`**. Yet another example where Linux basically allows you to do anything you want.


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651339395897/Bjj-NFXK1.png align="center") *("Source": https://9gag.com/gag/abVPE5O)*

Because I didn't brick the system and all I need is to change that line in the `sshd_config` - solution was simple. Mount that SD card in another Linux system and edit file.

Problem was that my main and only bare-metal system is Windows. So, I had to mount SD card on the guest Linux OS on a Windows host. Here is how

# Prepare hypervisor

To be able to see the SD card on the Linux inside VirtualBox you have to prepare VMDK ([Virtual Machine Disk](https://en.wikipedia.org/wiki/VMDK)) file linked to the device.

1. Put your SD card in a card reader (or USB adapter) and connect to the hardware.
2. Open PowerShell as an administrator. List available storage devices with `wmic diskdrive list brief`
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651340469231/UZMLLhKvd.png align="center")
3. Locate SD card `DeviceID` - in my example, it is `\\.\PHYSICALDRIVE3`.
4. Navigate to the VirtualBox installation folder (or execute command with absolute path) and create VMDK.
```ps1
.\VBoxManage.exe internalcommands createrawvmdk -filename D:\ETH\pi.vmdk -rawdisk \\.\PHYSICALDRIVE3
```
5. Run VirtualBox as administrator (file was created by the administrator, so a regular user won't be able to access it in VirtualBox). Note: VM should be turned off.
6. Open *Settings* for guest Linux machine. Navigate to *Storage*. Locate SATA Controller and enable *Use Host I/O Cache*.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651344440808/Tzq5cGFGq.png align="center")
7. Click small *Add hard disk* icon.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651344499448/dhxIHBFwZ.png align="center")
8. Click *Add*.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651344534152/HoTl6tCun.png align="center")
9. Browse for the VMDK file, select and confirm adding to the machine. You should see it is connected to SATA controller.

# Mount SD card

%%[join-cta]

1. Launch Linux host system (start VM).
2. List available devices. Find out which one is SD Card.
```txt
sudo fdisk -l
```
3. Create a mount point (if don't already created one for such purposes).
```txt
sudo mkdir /mount/external-sd
```
4. Mount file system.
```txt
sudo mount /dev/sdb2 /media/external-sd
```
> Don't mount whole device as `mount /dev/sdb ...`. Use only the partition that is not marked as boot (or more accuratly, mount Linux partition)
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651336463715/DNtwxinZn.png align="center")
5. Now SD card file system is available under `/media/external-sd`

# Safely remove SD card

1. After changes are made, unmount the device.
```txt
sudo umount /media/external-sd
```
2. Close the VM.
3. Remove VMDK file.
4. Safely remove hardware and eject media.

```sh
$ sudo mount /dev/sdb2 /media/external-sd
```

# Conclusion

%%[follow-cta]

After all that wasn't so problematic this time, becasue I had to modify back single line in SSH config.
..but now I have a second SD card with system that got closed abruptly during the updates. Now all my login tries got rejected 😅
