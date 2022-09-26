## How to modify the Windows Setup boot image

> I'm using Windows 11 *.iso, so details can be different on other versions - overall idea should be the same. This is being done for years to older images.

%%[join-cta]

# Process summary
This is three-phase process.
1. Extract ISO content.
* Modify `boot.wim` image.
* Build new **bootable** ISO containing the changes using external software (like [ImgBurn](https://www.imgburn.com/).

> ℹ It is important that we are creating the **bootable** media, as not any software can do that correctly.

# Phase 1

1. Prepare files you want to have access to during Windows installation process or note the files you want to change.
* Extract the content of the **.iso* Windows bootable media (ex. using 7zip).

# Phase 2

To modify the content of the environment you are booted to, you have to modify the *sources\boot.wim* file. This can be done using built-in **Deployment Image Servicing and Management tool** (`dism`). Read more about it [here](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/mount-and-modify-a-windows-image-using-dism?view=windows-11).

> ❗ Mounting images with DISM doesn't look like regular ISO mounting. It is more similar to Unix mounting style. You won't see additional drives, files are extracted to the directory, so it can be easy to forget about unmounting the image and start removing them by hand. Be careful then - some of the files will still be protected by the Windows, but when you accidently apply the changes later, you could break your WIM file.

1. Prepare empty directory where you will **mount** the image from *boot.wim* file.
* Open command prompt with Administrator rights and list the available images in **.wim* file.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663492342042/wMVd0svn-.png align="left")
>Windows PE (WinPE) is Windows Preinstallation Environment. Read more [here](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/winpe-intro?view=windows-11). You can think of this WIM file as a docker image with multiple layers. Here index 1 is the blank *layer* of WinPE and index 2 (or any last index) is the actual environment that is booted.
*  Mount the file using `dism` with last index of image found in *boot.wim*.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663493991002/Aa5DJ67EN.png align="left")
* Apply changes to the image. In my example I'm adding the REG file.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663494262060/9-79zfpmT.png align="left")
* Commit changes to the *boot.wim*.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663496212011/i7z5o5uOA.png align="left")
* Close any folders and software that may be using the mounted location. Otherwise, unmounting fails (which is not a failure, you can still correctly apply changes and unmount WIM, see the screens). 
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663494614046/osnYsDRie.png align="left")
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663494735928/LVRp7jqzo.png align="left")
* Now when you list available images from *boot.wim* you won't see another index, but you can see the difference in size. If you want to double check, you can mount the image once again, inspect the files and unmount.

%%[support-cta]

# Phase 3

1. Start ImgBurn (or any other software able to create bootable media).
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663361103181/o4oyNh73j.png align="left")
* Ensure *Build* mode is active. Choose source files by adding folder from phase one.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663361409544/X0DVP1lNV.png align="left")
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663361216214/9TSsSIoqA.png align="left")
* Now select destination (final ISO file).
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663361531183/0Iqam74q5.png align="left")
* Go to *Advanced *-> *Bootable Disc* settings and set the values as below.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663363605922/-IzpUK5M3.png align="left")
* Browse for the boot image (`efi\microsoft\boot\efisys.bin`)
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663363335925/F3yMylxeZ.png align="left")
* Now everything is read, so you can press the big image on the bottom left of the window to start creating the image file.

> ℹ During process ImgBurn will ask some questions - you can safely assume it is right and agree to changing some modes or adding labels to disc :).

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663495261398/O0thcW5oN.png align="center")

# Verify

1. Mount the patched ISO in virtual machine (or just live machine).
* Boot into Windows Installer.
* Launch Windows Setup Command and verify the changes are applied.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663495592421/EJJs2AFqu.png align="left")

# Additional readings

%%[follow-cta]

* [What determines the X:\Sources content of Windows Setup Command?](https://superuser.com/questions/1742840/what-determines-the-x-sources-content-of-windows-setup-command/1742851#1742851)
* [Modify a Windows image using DISM
](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/mount-and-modify-a-windows-image-using-dism?view=windows-11)