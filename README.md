# Windows on Linux

## Table of Contents:
1. [You Did What?](#what)
2. [Disclaimer](#disclaimer)
3. [Instructions](#instructions)

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
| Motherboard | Msi B450 Pro Carbon AC |
| CPU | AMD Ryzen 9 3900X |
| GPU | AMD Radeon RX 580 |
| Memory | G Skill Ripjaws 2x16Gb DDR4-3200 |
| SSD | SamsungEvo 1TB Nvme M.2 |

With that in mind, **make sure you understand exactly what is happening before following a step**, so you don't damage your machine. 

I'm also using an [Arch Linux](https://www.archlinux.org/)-based distribution, so I will be using [Pacman](https://wiki.archlinux.org/index.php/pacman) and [yay](https://github.com/Jguer/yay) to retrieve packages.

## Instructions <a name="instructions"></a>

**WIP**

