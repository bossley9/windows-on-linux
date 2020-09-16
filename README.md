# Windows on Linux
A step-by-step guide to playing games on a Windows 10 VM in Linux using PCI(e) passthrough with a single GPU in Qemu

> Note: This guide is not completed, and due to school committments, will not be completed for some time. I will work on finishing this whenever I can :)

## Table of Contents:
1. [About](#about)
1. [Who Is This Guide Intended For?](#who)
2. [Disclaimer](#disclaimer)
3. [You Did What?](#what)
4. [Instructions](#instructions)
5. [Resources](#resources)

## About <a name="about"></a>

I think one of the biggest defenses of Windows 10 over various Linux distributions is that _"Windows 10 can run games and Linux can't."_

Only normies would say this.

Playing games on Linux is only as complicated as you make it to be. The purpose of this project is to prove to all the normies out there that playing games on Linux is doable, even if the gaming industry doesn't completely back support for Unix-based systems (and to work around things like anticheat). 

> If you happen to be a Windows, macOS, or BSD user who has never touched Linux before, I suggest [trying Manjaro first](https://manjaro.org/downloads/official/gnome).

#### Solutions

First, there are many alternatives to gaming on Linux.

1. Find [open source games built with support for Unix](https://www.gamingonlinux.com/itemdb.php). The only downside is that, well... the selection of games is extremely limited.
2. Use a Windows DOS, Win32, or Windows executable runner like `Wine` or `DXVK`. Manually setting up Wine prefixes is a royal pain so I would suggest using an established client such as [GOG](https://www.gog.com/) or [Lutris](https://lutris.net/). Almost 99% of games can be found here.
3. Run a virtual machine. A virtual machine is creating a simulated environment in which you can run an operating system within a "black box". The core issue with gaming on a virtual machine is that a GPU can only output graphics to a single monitor (i.e. if you send all of your graphics to a virtual machine, your computer won't be able to display your desktop and your screen will turn off.

    A solution to this problem is single GPU [PCI(e) passthrough](https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF), in which the kernal passes the GPU output immediately from the host machine to the guest machine (the VM). Since the previous solutions are trivial in setup, this guide will focus on configuring single GPU PCI(e) passthrough.

## Who Is This Guide Intended For? <a name="who"></a>

This guide is intended for somewhat-experienced Linux users (preferably in Arch-based distros) who would like to run Windows programs which require advanced GPU processing on their machine (i.e. Steam games or graphic-intensive Windows programs). I'm also assuming you know nothing about virtual machines in Linux, `Qemu`, or `PCI(e) passthrough` for that matter.

## Disclaimer <a name="disclaimer"></a>

I hold no responsibility for damages done to your machine. This guide was tested on a machine with the following hardware and software:

| Hardware/Software | Specs |
| :--- | :--- |
| Operating System | Archlinux |
| Kernel | 5.8.5-arch1-1 |
| Window Manager | Bspwm |
| Motherboard | MSI B450 Pro Carbon AC |
| CPU | AMD Ryzen 9 3900X |
| GPU | AMD Radeon RX 580 |
| Memory | G Skill Ripjaws 2x16Gb DDR4-3200 |
| SSD | SamsungEvo 1TB Nvme M.2 |


With that in mind, **make sure you understand exactly what is happening before following any step**, so you don't damage your machine. I'll do my best to explain thoroughly the purpose of each step.

I'll also be using an [Archlinux](https://www.archlinux.org/)-based distribution so I will be using the [Pacman](https://wiki.archlinux.org/index.php/pacman) and [Yay](https://github.com/Jguer/yay) package managers to retrieve packages. You can see how to install Archlinux by following the guide in my [dotfiles](https://github.com/bossley9/dotfiles/blob/63e5e3d22f1e51eb7cd9ce7829e87a343d61cea2/README.md#manualinstall).

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

## Resources

Below is a list of useful resources I found which provided insight and helped me write this guide.

- [Arch Wiki: Using Identical Guest and Host GPUs](https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF#Using_identical_guest_and_host_GPUs)
- [Pavolelsig's Passthrough Helper for Manjaro](https://github.com/pavolelsig/passthrough_helper_manjaro)
- [Yuri Alek's VFIO Guide](https://gitlab.com/YuriAlek/vfio)
- [Karuri's VFIO Guide](https://gitlab.com/Karuri/vfio)
- [Qemu-KVM Introduction](http://alexander.holbreich.org/qemu-kvm-introduction/)
- [PCI Passthrough Via OVMF](https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF)
