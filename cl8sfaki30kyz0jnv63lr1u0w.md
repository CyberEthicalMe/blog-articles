## How to bypass Windows 11 requirements check


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663358982808/eFoIHU7L4.png align="center")

> ❗ Following steps have to be executed each time the installation media is booted. Registry edits are lost because installator doesn't persist its state between reboots.

%%[join-cta]

To bypass the requirements check (ex. when installing Windows on a virtual machine, purposely using less resources):

# Manual

1. Go back to the first screen.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663351875957/dwFlSOXU3.png align="center")

* Launch command line interface by pressing <kbd>Shift</kbd> + <kbd>F10</kbd>. Run `regedit`.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663352189990/5g1_WiP2O.png align="center")

* Expand `HKEY_LOCAL_MACHINE\SYSTEM\Setup`. RMB on that node to add new key `LabConfig`.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663352412748/S-YV3AqBO.png align="center")
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663352496768/J3GvRaBbr.png align="center")
* RMB on the right panel and add new "DWORD (32-bit)". Call it `BypassTPMCheck`. Double click and change value to `1`. Click "OK".
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663354143875/Wppj6BiSI.png align="center")
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663354264243/_orczt5uz.png align="center")
* Add three more values the same way: `BypassSecureBootCheck`, `BypassCPUCheck`, `BypassRAMCheck`. You should end up with the following setup:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663355050664/qmKs83k_r.png align="center")
* Close Registry Editor and continue with the installation.

%%[support-cta]

# Fancy

1. Create `*.reg` file with following content.
```reg
Windows Registry Editor Version 5.00
[HKEY_LOCAL_MACHINE\SYSTEM\Setup\LabConfig]
"BypassSecureBootCheck"=dword:00000001
"BypassTPMCheck"=dword:00000001
"BypassCPUCheck"=dword:00000001
"BypassRAMCheck"=dword:00000001
```
* Add this file to the Windows ISO installer using [this guide](https://blog.cyberethical.me/modify-windows-setup-boot-image-winpe).
* Boot machine with patched ISO

* On the first screen launch command line interface by pressing <kbd>Shift</kbd> + <kbd>F10</kbd>
* Type `regedit [/sources/*reg]` where `[/sources/*reg]` is the path to your REG file.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663495637380/upDKDTC3b.png align="left")
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1663495815651/UzIhs9I2V.png align="left")
* Close Registry Editor and continue with the installation.

# Additional readings

%%[follow-cta]

* [How to Bypass Windows 11's TPM, CPU and RAM Requirements](https://www.tomshardware.com/how-to/bypass-windows-11-tpm-requirement)