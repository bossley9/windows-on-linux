# Windows on Linux
A step-by-step guide for running a nearly native Windows 10 VM on Linux using Qemu, Libvirt and PCI(e)-passthrough with a single GPU

> Note: This guide is not completed, and due to school committments, will not be continued for some time. However, if you're looking for great resources on this topic, try the [Arch Wiki](https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF#Using_identical_guest_and_host_GPUs). You can also take a look at [Pavolelsig's passthrough helper](https://github.com/pavolelsig/passthrough_helper_manjaro) and [Yuri Alek's guide](https://gitlab.com/YuriAlek/vfio).

## Table of Contents:
1. [Who Is This Guide Intended For?](#who)
2. [You Did What?](#what)
3. [Disclaimer](#disclaimer)
4. [Instructions](#instructions)

## Who Is This Guide Intended For? <a name="who"></a>

This guide is intended for Manjaro/Arch Linux users who would like to run Windows programs which require advanced GPU processing on their machine (i.e. Steam games or graphics intensive Windows programs). 

## You Did What? <a name="what"></a>
**Running any sort of Windows program on Linux is hard**. It's almost physically painful to configure Wine, and for every game or executable you want to run, there's always a bunch of different arguments or flags you have to set that are specific to that program. As a casual gamer who enjoys Linux, it's especially aggravating when you're thinking about buying a new machine. 

_Should I just stick with Windows?_  
Of course you can - and sell your soul to Microsoft in the process. You've also just made any kind of software development or [ricing](https://www.reddit.com/r/unixporn/wiki/themeing/dictionary#wiki_rice) much more tedious and counter-productive. You might as well type code on a [Blackberry phone](https://blackberrymobile.com/us/).

_Well, I guess I can just dual boot..._  
Yes, and every time you need to run a Windows program or game, you can restart your computer. And then you accidentally forgot to fix a file in Linux so you need to restart again. Oops, you forgot to make a selection in the grub menu and it accidentally booted into the wrong boot option. Need I say any more?

_Fine, fine, I'll stick with Linux._  
No games for you.

> Actually, I've been using [Lutris](https://lutris.net/) for the past three months to game on Linux and it has amazing ports from Windows gaming to Linux using preset options in Wine, Proton and PlayOnLinux. But it's not perfect.  

And what if you need to run a program that doesn't work with Wine or Lutris, like [Visual Studio](https://visualstudio.microsoft.com/)? I encountered this very problem last month, which made me scramble to find a solution. Now that [COVID-19](https://en.wikipedia.org/wiki/Coronavirus_disease_2019) is forcing everyone to work remotely, I can't just use my work machine to run Visual Studio, and our remote desktop service has terrible performance. 

So, of course, I chose to run a virtual machine. But there's still a problem - any virtual machine creates an emulated GPU, meaning it won't use your machine's native GPU. This limits the number of games and programs you can run on your virtual machine to about zero - unless all you wanted to play was MS-DOS games from the 90's. 

This is something I couldn't compromise on. My work uses [Unreal Engine 4](https://www.unrealengine.com/en-US/) for game development, which requires a decent GPU to run. How do you tell a virtual machine to use a hardware GPU?

One solution is [PCI(e) passthrough](https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF). Essentially, the virtual machine uses a secondary hardware GPU and passes it to the guest, which uses it to produce near-native performance. The only problem was that my Linux machine did not have two GPUs, and I refused to spend the money to buy another GPU just for Windows programs.

After much researching and testing, I found a solution which utilizes PCI(e) passthrough with only one GPU and still provides near-native performance. This is a step-by-step guide on how to configure this for a Linux machine.

## Disclaimer <a name="disclaimer"></a>

I hold no responsibility for damages done to your machine. This guide was tested on a machine with the following hardware and software:

| | |
| :--- | :--- |
| Operating System | Manjaro Linux 19.0.2 Kyria |
| Kernel | 5.4.24-1-MANJARO |
| Window Manager | Gnome |
| Motherboard | Msi B450 Pro Carbon AC |
| CPU | AMD Ryzen 9 3900X |
| GPU | AMD Radeon RX 580 |
| Memory | G Skill Ripjaws 2x16Gb DDR4-3200 |
| SSD | SamsungEvo 1TB Nvme M.2 |


With that in mind, **make sure you understand exactly what is happening before following any step**, so you don't damage your machine. 

I'm also using an [Arch Linux](https://www.archlinux.org/)-based distribution, so I will be using [Pacman](https://wiki.archlinux.org/index.php/pacman) and [yay](https://github.com/Jguer/yay) to retrieve packages.

## Instructions <a name="instructions"></a>

1. Virtualization needs to be enabled before anything. You won't be able to run a virtual machine without it. To enable virtualization in the kernel, you will need to change your BIOS settings to enable virtualization. Each BIOS has different settings. You can verify virtualization is enabled by running `egrep "svm|vmx" /proc/cpuinfo`. If the command outputs a list of flags, virtualization is enabled.

2. Install `qemu`, `ovmf`, and `libvirt`.
    ```
    sudo pacman -S qemu ovmf libvirt
    ```
    If the `ovmf` package is unavailable in all repositories (like it was for me), you can also install the AUR version:
    ```
    yay -S ovmf-git
    ```
3. Enable IOMMU. Edit `/etc/default/grub` and add `amd_iommu=on` and `iommu=pt` to `GRUB_CMDLINE_LINUX_DEFAULT`. Note that these modules may be different depending on your architecture.
	  ```
	  GRUB_CMDLINE_LINUX_DEFAULT="amd_iommu=on iommu=pt"
	  ```
	  Then regenerate grub.
	  ```
	  sudo grub-mkconfig -o /boot/grub/grub.cfg
	  ```
	  Reboot. Verify that IOMMU works with `sudo dmesg | grep -e DMAR -e IOMMU`.
    If you want to double check that IOMMU works, try running the `iommu.sh` script in this repository, which will show the different IOMMU groups if IOMMU is set up properly ([script source](https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF#Prerequisites))
4. You'll need to download a Windows 10 disc image. Microsoft provides a [free download](https://www.microsoft.com/en-us/software-download/windows10ISO), but you'll likely need an activation key.
5. Start the `libvirt` daemon.
  	```
  	sudo systemctl start libvirtd
  	```
    You can verify this is running with `sudo systemctl status libvirtd`. If it says "Failed to initialize a valid firewall backend", you will need to install additional packages, then restart `libvirtd`.
    ```
    sudo pacman -Syu ebtables dnsmasq
    sudo systemctl restart libvirtd
    ```
6. Next, you will need to get the ids of the gpu display and gpu audio. Run the following command, which will display input similar to mine.
    ```
    lspci -nnk | grep VGA -A 7

    ----------------------------------------------------------------

    26:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Ellesmere [Radeon RX 470/480/570/570X/580/580X/590] [1002:67df] (rev e7)
      Subsystem: Tul Corporation / PowerColor Ellesmere [Radeon RX 470/480/570/570X/580/580X/590] [148c:2378]
      Kernel driver in use: amdgpu
      Kernel modules: amdgpu
    26:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Ellesmere HDMI Audio [Radeon RX 470/480 / 570/580/590] [1002:aaf0]
      Subsystem: Tul Corporation / PowerColor Ellesmere HDMI Audio [Radeon RX 470/480 / 570/580/590] [148c:aaf0]
      Kernel driver in use: snd_hda_intel
      Kernel modules: snd_hda_intel
    ```
7. Before you proceed past this step, **make sure** you have saved and closed every window before continuing.
	  Kill the display manager. This will vary depending on your display manager. This will kill X.
	  ```
	  systemctl stop gdm
    ```

**WIP**
