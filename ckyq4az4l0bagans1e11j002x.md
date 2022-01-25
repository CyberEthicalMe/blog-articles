## Kasm Workspaces: Spinning up containers on the Raspberry Pi

> Raspberry Pi Kasm Guide was inspired by [NetworkChuck](https://www.youtube.com/watch?v=U7e-mcJdZok), go watch this and his other videos, he's living inspiration ☕

# Prerequisites

I am running the following setup, but feel free to test on other configurations.
>I'm getting a small commision for the purchases made from the links below

* [Raspberry Pi 4B 8GB RAM](https://amzn.to/3GSMxTV)
* [SD Card (at least 64GB)](https://amzn.to/3GXjw9S)
* [Raspberry Pi 4 Power Supply](https://amzn.to/3KrMzEt)
* [Argon NEO Raspberry Pi 4 Case](https://amzn.to/3tLgn9l)
* [Ethernet cable (for the initial configuration or Internet over LAN)](https://amzn.to/3fLdnkS)

# Prepare Ubuntu Server SD Card

1. Download Ubuntu Server 20.04 LTS for Raspberry Pi 4 from the [official download page](https://ubuntu.com/download/raspberry-pi).
![Screenshot_43.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642608877580/9sJc4cMwb.png)
2. Verify hash (should match)
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642611806308/UoFmv9YbW.png)
> On Windows you can get SHA256 sum from PowerShell
> ```sh
# PowerShell
Get-FileHash -Algorithm SHA256 .\ubuntu-20.04.3-preinstalled-server-arm64+raspi.img.xz
```

3. Run [Raspberry Pi Imager](https://www.raspberrypi.com/software/) or [Etcher.IO](https://www.balena.io/etcher/) to write Ubuntu to the SD Card.
> Unfortunately this distro [doesn't work](https://github.com/raspberrypi/rpi-imager/issues/213) with Advanced features of Raspberry Pi Imager (<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>X</kbd>). Raspberry just won't boot if any of advanced options is used.
4. When the process ends, insert a SD Card, connect power and wait a bit for the device to boot. Then SSH into the Ubuntu and start post-install steps.

# Optional post-install steps

The following steps are **optional**, but **highly recommended** from a hardening perspective.

>Additonally I have my own shell profile that I can download from the GitHub whenever I know I will be using Linux instance for a longer while.
```sh
$ wget https://github.com/KamilPacanek/configs/raw/master/kali-rearm/.shell_aliases -O ~/.bash_aliases
$ source ~/.bash_aliases
```

1. Login via SSH ([default credentials](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#4-boot-ubuntu-server) `ubuntu:ubuntu`) and change password to more secure one - we will be using it later for `sudo` commands.
* As we didn't have possibility to change username or hostname during the system installation, we have option to do it now. Change hostname (`/etc/hosts` and `/etc/hostname`, then reboot).
* Now perform following steps from [Hardening](https://blog.cyberethical.me/how-to-install-kali-on-a-raspberry-pi#heading-hardening) section: *Add new suduer account*, * Enable SSH login*, *Disable root & password login*, *Remove default user*.
* Add the following aliases to your `.bash_aliases`. These are easier to call than remembering the paths.
```sh
alias kasm-start="sudo /opt/kasm/bin/start"
alias kasm-stop="sudo /opt/kasm/bin/stop"
```

# Mandatory post-install steps

1. Relogin on new user using SSH and update packages and distribution (see the `apt-updater` from my `.shell_aliases`)
* As you probably will be exposing Kasm to the Internet, it's best to have the important patches applied both manually and automatically. For the second part, I recently started using the `unattended-upgrades` [package](https://wiki.debian.org/UnattendedUpgrades).
* Ensure swap partition is available. Run `$ cat /proc/swaps` to see if any swap is already created. In my case, it isn't, so I'm running this command to create and mount 4GB of swap.  
*Kasm recommend [1GB per concurrent session](https://www.kasmweb.com/docs/latest/install/single_server_install.html#creating-a-swap-partition) and I don't plan to run more than two images simultaneously. I have plenty space on my SD Card, so 2GB buffor sounds reasonable with my 8GB RAM on Raspberry.*
```sh
# don't bother tinkering with 'bs' and 'count' arguments;
# did a test with 1M/4096 and 512K/8192 and both completed at same time
$ sudo dd if=/dev/zero bs=1M count=4096 of=/mnt/4GiB.swap
$ sudo chmod 600 /mnt/4GiB.swap
$ sudo mkswap /mnt/4GiB.swap
$ sudo swapon /mnt/4GiB.swap
# make swap available on boot
$ echo '/mnt/4GiB.swap swap swap defaults 0 0' | sudo tee -a /etc/fstab
```

# Install Kasm Workspaces

%%[support-cta]

1. Download Kasm Workspaces from the [official download page](https://www.kasmweb.com/downloads.html).
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642852860065/nX9ejRX7x.png)
```sh
$ cd /tmp
$ wget https://kasm-static-content.s3.amazonaws.com/kasm_release_1.10.0.238225.tar.gz
# hash shoud match the one visible on download page
$ sha256sum kasm_release_*.tar.gz
```
* Now is the time to decide on which port you would like the Kasm Web UI to work on. This can be changed later, but it is easier now. Default is 443.
```sh
$ tar -xf kasm_release*.tar.gz
# sudo bash kasm_release/install.sh -L 8443 / change default port
$ sudo bash kasm_release/install.sh
```
Wait for install to finish. When something goes wrong, just restart the `install.sh`.
> Irrelevant for us here, but worth noting: if you are running Kasm from VM connected via NAT with port forwarding - ports on host and client should match. Otherwise, you would get an error when connecting to the Kasm images - internal connection will try to reach out to Kasm port. You can either catch that connection in the Dev Tools and manually add forwarded port or [setup port on Kasm](https://kasmweb.com/docs/latest/how_to/port_change.html).
* Copy the displayed credentials and save them for future use.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642806723923/rBRwV1mBz.png)

# Fix Kasm Web UI SSL error

This topic is an unexpected addition to this guide. When you first enter the `https://{RASPBERRY_IP}/` in the browser, you'll see the `ERR_CERT_AUTHORITY_INVALID` error.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642868675457/54txx7ECR.png)

And that's completely accurate behaviour because during the installation, Kasm is generating self-signed certificate for the hostname address.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642868939881/L6INK1X0W.png)

> In the next guide I will show how to deal with it in more elegant manner, but for now let's continue.

What I'm proposing as a temporary solution is to install the existing Kasm certificate in the Trusted Root Authorities and add `hosts` entry, so the Kasm address domain matches the certificate domain.

1. SSH into Kasm server (Raspberry Pi). Copy Kasm certificates and change their owner to current user.
```sh
$ mkdir ~/kasm_certs
$ cd ~/kasm_certs
# we have to sudo copy them because of their 600 permissions
$ sudo cp /opt/kasm/current/certs/* .
# now change owner of the copied files
$ sudo chown {LOGIN} *
```

2. Secure copy the certificates to the local system (executed on the system that is going to use the Kasm; endpoint)
```sh
$ scp -r -i {PEM_KEY_PATH} {USER}@{RASPBERRY_IP}:/home/{USER}/kasm_certs .
```
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642869692816/9eIfoA63f.png)

3. Double-click the `kasm_nginx.crt` and add it to the Root Trusted Certificates of the Local Machine.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642869869233/cA-ieIygq.png)

4. Open Notepad **as an administrator**. Open `C:\Windows\System32\drivers\etc\hosts`. Add following line
```txt
{RASPBERRY_IP} kasm-pi
```

5. Now clear browser cache and navigate to the `https://kasm-pi/`. Now you can still get the error because Common Name for the self-signed certificate is empty, but you **can ** use the Kasm.

# Kasm overview

%%[join-cta]

Navigate to the Kasm Web UI.

> Kasm can be stopped and started from the server with the following commands
```sh
sudo /opt/kasm/bin/stop
sudo /opt/kasm/bin/start
```

## Change default passwords

Navigate to the Kasm Web UI.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642867563314/VTt40l_KI.png)

>Don't mind the round key icon on the input field, that's the [KeePassXC Chrome extension](https://keepassxc.org/docs/KeePassXC_GettingStarted.html#_install_the_browser_extension)

Now login with the `admin@kasm.local`For the security reasons you should use Kasm as a normal user, not admin - but this time we are logging in to change the default passwords. Go to Users on the Admin panel.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642870478547/ErHYUTun3.png)

Click on the ellipsis next to each user and assign a new password using "Reset password" option.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642870546935/w3CwgM5k-.png)

Relogin as an admin.

## Images
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642870663981/CfUG1dLr1.png)

Here you can see all the images that can be spun on the Kasm. You can add more from the [Kasm's Docker Hub](https://hub.docker.com/u/kasmweb). This possibility will be researched in the future guides. 

## Dashboard
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642870832062/gkPSHpVH7.png)

Here you can get some statistics of errors and resources available on the server. I encourage running `htop` on the server to see how lunching new Kasm images affect CPU and memory usage.

You will also see here the current sessions - useful place to terminate them.

## Workspaces

Finally, workspaces. Login as `user` and click on some tile to launch the new session. This will spun up the new docker instance on the server and will stream that to the browser tab.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642871072427/VF231WXJnh.png)

This is an isolated instance of that application, that can (and should be) terminated afterwards.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642871137352/cf1XMbPlq.png)

# Conclusion

You have now a working Kasm instance, ready to launch an isolated application, streamed to your browser. In the next articles we will address the certificate issue, add new images to the Kasm, manage persistence between sessions and more!

%%[follow-cta]