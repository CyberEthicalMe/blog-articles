## How to install Kali on the Virtual Machine

# Download system image

%%[support-cta]

Go to [Kali download page](https://www.kali.org/get-kali/#kali-bare-metal). For this guide, I'm choosing the bare-metal option because I used to this form of installation, and it gives me always a bit more control over the process.

![2021-08-29-10-30-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257642470/LmuZPPQF6.png)

This way, we will be proceeding as if we have an installer disk and a clean workstation without OS installed.

## Validate checksum

![kali-bare-sum.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257652432/hDN8N0-p3.gif)

We are going to deal with an offensive security oriented system, so it is especially **crucial** to verify that the file was not corrupted or altered in any way. Follow the steps described in [How to install Kali on a Raspberry Pi: Download image](https://blog.cyberethical.me/how-to-install-kali-on-a-raspberry-pi#download-image).

# Create new VM

> There are various option when it comes to dealing with virtual machines. I like to use [Virtual Box](https://virtualbox.org/) because it is free, not limited to number of machines and never failed me.

On the main window, click _New_.

> If you don't see this button, you can also choose _Machine_ -> _New_ from the menu.

![2021-08-29-10-26-28.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257662147/I_ebfu1KU.png)

Because Kali is a system [based on a Debian distribution](https://www.kali.org/docs/introduction/what-is-kali-linux/) we can choose the appropriate preset from a dropdown.

> Name of the machine can be changed later, but folder where the machine is located will stay the same. It can be moved to the different folder (ex. that correspondes to the new VM name) but it is out of scope of this guide.

![2021-08-29-10-43-56.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257671322/zGkvYnoV7.png)

[Official recommendation](https://www.kali.org/docs/installation/hard-disk-install/) for this kind of install is at least 2 GB of RAM and 20 GB of disk space.

![2021-08-29-10-56-09.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257680701/LsMehsISu.png)
![2021-08-29-10-56-24.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257686566/r-QS-s8zd.png)

You can choose the disk file by yourself - I tend to use `VDI` for no specific reason.

![2021-08-29-10-56-40.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257696097/dkwTuqRvr.png)

Now. Because I know I will be extending the disk in the future I can choose the **dynamic allocation** right now, but if you already have the fixed size disk - it [can be converted](https://www.howtogeek.com/312456/how-to-convert-between-fixed-and-dynamic-disks-in-virtualbox/) to the dynamic (or more accurately it can be cloned as dynamic).

![2021-08-29-11-04-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257701588/AFLtTqpwH.png)

Dynamic allocation has a nice property that you can specify its size that will be **seen**, but the **real** size that `VDI` file takes space is the actual usage of the disk. If it is not clear right now, it will be later.

Because we are following the recommendation, let's select the 20GB disk space.

> If you don't want to resize the disk later, choose 50GB.

![2021-08-29-11-07-47.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257712192/a2X30LmWO.png)

Now VirtualBox is preparing everything, and we have our machine visible on the list of available boxes.

![2021-08-29-11-09-16.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257723082/CCq52d7MIw.png)

You can ask a question: "we have chosen the RAM, the disk - what about CPU cores?". And it is a good question. Virtual machines tend to be flexible, so at any time, the machine is not powered on, you can specify these parameters in the machine settings. Go to _Settings_ → _System_ and select **at least 2 cores** - I find that my setup works better at 4 cores.

![2021-08-29-11-11-34.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257735424/NraLCR216.png)

# Network interfaces

This topic is so wide that it deserves a separate article. You can find a great elaboration on the [Nakivo blog](https://www.nakivo.com/blog/virtualbox-network-setting-guide/). I want to have an Internet access from this VM, so I'm leaving the default NAT.

# Install Kali from image

Because we don't have any boot information saved on the disc and nothing mounted in the IDE controller, when you launch the VM right now you will see the popup to choose boot disk and if continue to cancel we won't go further with that.

![2021-08-29-11-26-01.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257742051/E2UAWy0IR.png)

Select the downloaded Kali image installation media by going to _Settings_ → _Storage_, and mount the `*.iso` in the empty IDE device.

![2021-08-29-11-28-53.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257753046/j1x3OHVbh.png)

Now when you launch the machine you will see the installation menu.

![2021-08-29-11-30-53.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257760911/esfn8fDHI.png)

I choose the _Graphical install_.

> Don't worry about mount fails - these are [completely normal phenomenon](https://knowyourmeme.com/memes/anatoly-dyatlov).

Next steps are pretty straightforward.

![kali-bare-install.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257779274/Z9lXcS5SW.gif)

%%[join-cta]

## Choose partitioning method

This is a frequent topic of discussions - _should I use LVM or not_. Simple answer - yes, you want. Longer and more correct answer - **it depends**. A significant advantage of LVM is ease of repartitioning with minimal (but still significant on heavily exploited system) overhead. 

> I have collected more detailed LVM analysis in [Additinal readings](#additional-readings) section

As you can see, there are two options - `LVM` and `LVM encrypted`.
![2021-08-29-13-07-29.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257788392/TgIO_LjAV.png)

In my opinion, the encryption option has more sense when you are installing Kali on an actual device not the VM (of course, unless you run VM on a laptop). In case your physical device got stolen, your data is most probably safe.  
The second thing to consider is what this system will be used for. If you are planning to store there some sensitive data, especially external party ones (for example results of your bug bounty hunting) remember that you are responsible for securing that information from the unprivileged eyes. Encrypting the disc requires you to remember and enter an additional password each time the VM is booted. There is also some (little) performance hit on I/O operations, as would be expected, because data are encrypted on the fly.

For the purpose of this guide, I will choose **encrypted LVM**.

![2021-08-29-14-00-30.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257798893/ZspQf5aWy.png)

Generally, separated partitions are recommended because
* you can upgrade or change distribution without losing your data from `/home`
* having `/var` and `/tmp` at separate partitions prevent `/` from filling up (ex. in case malicious process is running)
* separated partitions can be mounted as read-only, where otherwise it would be difficult to achieve

In the next window, select `Yes` if you are satisfied with the settings and let the installer create your LVM partition.

Because I have chosen encrypted disk, I am prompted to enter the 20+ character long passphrase. I'm using my [KeePassXC](https://keepassxc.org/) to generate a strong enough phrase.
![2021-08-29-14-26-12.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257806857/QXatXk5gx.png)

I am using the whole available space
![2021-08-29-14-32-05.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257812519/eb2n4QCFJ.png)

And finally leave _Finish partitioning_ to write changes to disk.
![2021-08-29-14-33-31.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257824004/ttMre-3I1.png)

## Additional software

![2021-08-29-14-46-46.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257818359/TsaWfxfu4.png)

I'm a fan of installing only software I need, but here to save time, I can consider choosing `large`.

> During the installation this time I've got the error on this step. Tried to fix that, but finally I've chosen the _Abort installation_ option and tried without choosing the `large` tools pack. Please let me know in the comments if it was the same case for you.

Follow screenshots to finish installation.

![2021-08-29-16-05-25.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257836226/dBc_-_N_N.png)
![2021-08-29-16-08-58.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257842828/CjmMPT-AX.png)
![2021-08-29-16-11-28.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257848218/wg54cwAid.png)

# Last steps

![2021-08-29-16-12-57.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257853423/9dikQNRTA.png)

Now, when you boot into the system, you have to unlock the drive first (if you have chosen the encrypted drive option). After that, we can check for example how installation process partitioned our directories.

![2021-08-29-16-17-39.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257866969/qCfixv-ad.png)

As we can see, it doesn't look good - 78% of root partition already used and even without running initial updates. Let's see how it will look afterwards.

![2021-08-29-16-31-28.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257874423/epLl-8ULS.png)

And we hit the limit. In the next article, I'll show how to resize disk.

For example, this is how it looks on my 5 month's personal installation:
![2021-08-29-16-40-55.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630257879448/huB5ubOHP.png)

# Additional readings

* [Kali installation on AWS EC2](https://blog.cyberethical.me/kali-linux-on-amazon-ec2)
* [Kali installation on Raspberry Pi](https://blog.cyberethical.me/how-to-install-kali-on-a-raspberry-pi)

%%[follow-cta]

* [VirtualBox Network Settings: Complete Guide](https://www.nakivo.com/blog/virtualbox-network-setting-guide)
* [When to use LVM](https://blog.vpscheap.net/when-to-use-lvm/)
* [Logical Volume Manager (LVM) versus standard partitioning in Linux](https://www.redhat.com/sysadmin/lvm-vs-partitioning)
* [Pros and cons of encrypted LVM](https://www.reddit.com/r/debian/comments/iyxz9s/pros_and_cons_of_encrypted_lvm/)