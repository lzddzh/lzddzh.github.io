---
title: 'Build Linux Kernel'
layout: post
tags:
  - Linux Kernel
  - tutorial
category: Linux
---

In this report, we will describe the whole workflow of installing a specific Linux kernel on Ubuntu16.04 at the Virtual Machine environment. Beyond the default compile configuration of Linux kernel, we will try to install a customized kernel with the minimal kernel, which will cost less time and disk space on compiling. 

<!--more-->
## 0. Environment
### Host Machine
**Machine:** MacBook Pro (13-inch, Late 2016, Four Thunderbolt 3 Ports)  
**Processor:** 3.1 GHz Intel Core i5 (2-core)  
**Memory:** 16 GB 2133 MHz LPDDR3  
**Operating System:** MacOS Sierra10.12.3  


### Guest Machine Settings
**Software:** Virtual Box 5.1.15, VMware 8.5.3 (just one of them is enough)  
**CPU Core Number:** 2  
**Memory Size:** 2GB  
**Disk Size:** 35GB  
**Operating System:** Ubuntu16.04.1 LTS **Kernel:** 4.4.0-31-generic   
**GCC:** 5.4.0

## 1. Build Kernel
### Install Virtual Machine and Ubuntu16.04
It is pretty easy to download the Virtual Box and install it. No need for me to give unnecessary details in this section. Just show you the screen shot of the Ubuntu16.04 inside the virtual box as below.

I found that at the creating virtual machine stage, if we choose the `vmdk` format instead of `vdi` format of the virtual disk file, then we can also run the same guest image on a VMware. And on my computer, the VMware is much faster than Virtual Box. Since what we did under VMware can also open in Virtual Box, I did all the below tasks under VMware, which provided me a smooth and enjoyable virtual machine experience.
![ubunutu16.04 info]({{site.url}}/assets/img/postImgs/2017-02-10-ComplieTheLinuxKernel/ubuntuInfo.png)

### Configuration
After reading the README file provided by Linux kernel 4.6, we now know that before we compile our Linux kernel, we should configure the compiling process by a `.config` file, which contains the key-value pairs to setting whether a specific module will be included in the kernel we build.

Using below command we are able to configure the `.config` file in a GUI way:  

```
$ make menuconfig
```

But after you input the command in terminal, errors occurs:

```
In file included from scripts/kconfig/mconf.c:23:0:
scripts/kconfig/lxdialog/dialog.h:38:20: fatal error: surses.h No such file or directory
```
This is because the menuconfig needs the `ncurses` libraray to generate its graphcial user interface. `ncurses` is a programming library providing an application programming interface that allows the programmer to write text-based user interfaces in a terminal-independent manner. It is a toolkit for developing "GUI-like" application software that runs under a terminal emulator. To install it, run:

```
$ sudo apt-get install libncurses5-dev
```

Now we could see the GUI-like interface in the terminal by using `make menuconfig`. Since this time we only aim to let the kernel work, we just let the default settings and save the `.config` file.

###Compiling the Source
To compile the source code of Linux kernel, run:

```
$ make -j2
```
The `-j2` tells the compiler to use 2 CPU cores to parallel the compiling process so it could be faster. 

Again, an error occurs as follows:

```
scripts/extract-cert.c:21:25: fatal error: openssl/bio.h: No such file or directroy
```

`SSL` is a module that provides security network encryption. The kernel build process needs this library. So install it by:

```
$ sudo apt-get install libssl-dev
```
After this was installed, I began my kernel compiling successfully.
On my computer, it cost about **50 minutes** to finish the making. And generated about **15GB** disk files.


### Install the Kernel

By running the below commands, we can install the binary executables we compiled from source into the ubuntu system, including the kernel and the kernel modules.

```
make modules install
make install
```
**Edit the Grub.cfg:**
After the above command, I find that the `grub.cfg` under `/boot/grub/grub.cfg` has already been changed to load the new kernel. So there is no need for us to edit it. But we can still see where dose it changed. By comparing with an old `grub.cfg` file, I find the changed lines:

```
$ diff --unchanged-line-format="" /boot/grub/grub.cfg grub.cfg | grep 4\\.6
```
Use the above command we will see the difference of `grub.cfg`:

![grubChange]({{site.url}}/assets/img/postImgs/2017-02-10-ComplieTheLinuxKernel/grubChange.png)

Basically, we can find in the screen shot that the kernel image 4.6.3 was added into the grub, I think the key line here is:

```
$ initrd /boot/initrd.img-4.6.3
```

### Restart the Ubuntu16.04
Now every thing is prepared and we just restart our operating system. 

Fortunately! my first try succeed!! Our new kernel is up:

![newKernel]({{site.url}}/assets/img/postImgs/2017-02-10-ComplieTheLinuxKernel/newKernel.png)

We can see the kernel version by using command `uname -r`, and as the screen shot shows, the Ubuntu16.04 is now starting up with its new kernel 4.6.3.  



## 2. Includes, Excludes and Modularize
In the **Configuration** section, we have been asked to select the configuration to wether `includes`, `excludes` or `modularize`. This is because in the source code of Linux kernel, except for the Linux operating system code, there are some other important programs to the system, such as drivers, network support, etc. 

These programs are called **Modules** of the Linux kernel. For a given computer hardware, usually there is no need to install all the provided modules provided. So the user can decide which modules will be installed at the configuration stage. And this is what exactly we did in the configuration section that saves a `.config` file from the GUI-lick interface.

Besides, the user can also choose whether to 'put the module into the kernel image' or just build as a 'loadable kernel module'. The loadable kernel module in fact is an object file that can be dynamically called by the kernel.

**For short answer:** 

- The `includes` option will put the module into the kernel image.
- The `excludes` option will not compile the module.
- The `modularize` option will make the module as a loadable module and will not be put into the kernel image.

![menu]({{site.url}}/assets/img/postImgs/2017-02-10-ComplieTheLinuxKernel/menu.png)

## 3. Smallest Kernel Image

So far so good. But when we see the size of kernel we built:

```
$ cd /boot
$ ls -la | grep init
```

And we get:
![kernelImg]({{site.url}}/assets/img/postImgs/2017-02-10-ComplieTheLinuxKernel/kernelImg.png)

From the above screen shot we can see that the size of our kernel is **297M**, which is much bigger than the initial kernel **35M**.

**Why does this happen?**
This is because after we run the command `make menuconfig` in the configure stage, we just leave the default options of the modules selection, which means we installed too many unnecessary modules in our kernel. 

**How to decide whether a module should be included ?** The answer differs from computers to computers. One important part of modules are the drivers that drive your computer hardware. Each computer has different hardware, so you need to see your hardware information of your computer. For example, you may see your sound card information to decide whether you need a sound card driver module in your kernel.

**Is there an automatical way to have a minimal configuration?** Since there are too many of modules to select, one could exhaust in doing such thing. Fortunately, when we read the `README` given in the kernel source code folder, we find this magic command:

###Select Minimum Number of Modules
When you are not familiar with some software, the best to do is to read the README. We find here is a possible useful make option for us. After some search on the keyword `localmodconfig`, I confirmed this idea.  

**Linux Kernel 4.6.3 REAME[^linux documentation] line 205:**

```
...
205 "make localmodconfig" Create a config based on current config and
206                      loaded modules (lsmod). Disables any module
207                      option that is not needed for the loaded modules.
...
```
After some googling, we know that:

- Localmodconfig usually finds the distribution kernel's basic configuration file automatically because all major distributions store it as `/boot/config-$(uname -r)` If another file is to be used as a starting point, copy it as ".config" to the top level directory of the decompressed kernel sources.[^quote]

- The localmodconfig will only load the modules that current system is using, in other words, the module in `lsmod`.[^quote1]

According to the above explanations, we just need to coyp the ubuntu release version config file to the root folder of the source code and run make localmodconfig:

```
$ cd linux-4.6.3
$ make localmodconfig
```
Then it will show up some selected or not selected modules:

**List of Modules Selection:**

```
zhendongliu@zhendongliu-virtual-machine:~/Desktop/mini/linux-4.6.3$ make localmodconfig
  HOSTCC  scripts/basic/fixdep
  HOSTCC  scripts/kconfig/conf.o
  SHIPPED scripts/kconfig/zconf.tab.c
  SHIPPED scripts/kconfig/zconf.lex.c
  SHIPPED scripts/kconfig/zconf.hash.c
  HOSTCC  scripts/kconfig/zconf.tab.o
  HOSTLD  scripts/kconfig/conf
using config: '/boot/config-4.6.3'
*
* Restart config...
*
*
* PCI GPIO expanders
*
AMD 8111 GPIO driver (GPIO_AMD8111) [N/m/y/?] n
BT8XX GPIO abuser (GPIO_BT8XX) [N/m/y/?] (NEW) m
Intel Mid GPIO support (GPIO_INTEL_MID) [Y/n/?] y
OKI SEMICONDUCTOR ML7213 IOH GPIO support (GPIO_ML_IOH) [N/m/y/?] n
RDC R-321x GPIO support (GPIO_RDC321X) [N/m/y/?] n
*
* PCI sound devices
*
PCI sound devices (SND_PCI) [Y/n/?] y
  Analog Devices AD1889 (SND_AD1889) [N/m/?] n
  Avance Logic ALS300/ALS300+ (SND_ALS300) [M/n/?] m
  Avance Logic ALS4000 (SND_ALS4000) [N/m/?] n
  ALi M5451 PCI Audio Controller (SND_ALI5451) [N/m/?] n
  AudioScience ASIxxxx (SND_ASIHPI) [N/m/?] n
  ATI IXP AC97 Controller (SND_ATIIXP) [N/m/?] n
  ATI IXP Modem (SND_ATIIXP_MODEM) [N/m/?] n
  Aureal Advantage (SND_AU8810) [N/m/?] n
  Aureal Vortex (SND_AU8820) [N/m/?] n
  Aureal Vortex 2 (SND_AU8830) [N/m/?] n
  Emagic Audiowerk 2 (SND_AW2) [N/m/?] n
  Aztech AZF3328 / PCI168 (SND_AZT3328) [N/m/?] n
  Bt87x Audio Capture (SND_BT87X) [N/m/?] n
  SB Audigy LS / Live 24bit (SND_CA0106) [N/m/?] n
  C-Media 8338, 8738, 8768, 8770 (SND_CMIPCI) [N/m/?] n
  C-Media 8786, 8787, 8788 (Oxygen) (SND_OXYGEN) [N/m/?] n
  Cirrus Logic (Sound Fusion) CS4281 (SND_CS4281) [N/m/?] n
  Cirrus Logic (Sound Fusion) CS4280/CS461x/CS462x/CS463x (SND_CS46XX) [N/m/?] n
  Creative Sound Blaster X-Fi (SND_CTXFI) [N/m/?] n
  (Echoaudio) Darla20 (SND_DARLA20) [N/m/?] n
  (Echoaudio) Gina20 (SND_GINA20) [N/m/?] n
  (Echoaudio) Layla20 (SND_LAYLA20) [N/m/?] n
  (Echoaudio) Darla24 (SND_DARLA24) [N/m/?] n
  (Echoaudio) Gina24 (SND_GINA24) [N/m/?] n
  (Echoaudio) Layla24 (SND_LAYLA24) [N/m/?] n
  (Echoaudio) Mona (SND_MONA) [N/m/?] n
  (Echoaudio) Mia (SND_MIA) [N/m/?] n
  (Echoaudio) 3G cards (SND_ECHO3G) [N/m/?] n
  (Echoaudio) Indigo (SND_INDIGO) [N/m/?] n
  (Echoaudio) Indigo IO (SND_INDIGOIO) [N/m/?] n
  (Echoaudio) Indigo DJ (SND_INDIGODJ) [N/m/?] n
  (Echoaudio) Indigo IOx (SND_INDIGOIOX) [N/m/?] n
  (Echoaudio) Indigo DJx (SND_INDIGODJX) [N/m/?] n
  Emu10k1 (SB Live!, Audigy, E-mu APS) (SND_EMU10K1) [N/m/?] n
  Emu10k1X (Dell OEM Version) (SND_EMU10K1X) [N/m/?] n
  (Creative) Ensoniq AudioPCI 1370 (SND_ENS1370) [N/m/?] n
  (Creative) Ensoniq AudioPCI 1371/1373 (SND_ENS1371) [M/n/?] m
  ESS ES1938/1946/1969 (Solo-1) (SND_ES1938) [N/m/?] n
  ESS ES1968/1978 (Maestro-1/2/2E) (SND_ES1968) [N/m/?] n
  ForteMedia FM801 (SND_FM801) [N/m/?] n
  RME Hammerfall DSP Audio (SND_HDSP) [N/m/?] n
  RME Hammerfall DSP MADI/RayDAT/AIO (SND_HDSPM) [N/m/?] n
  ICEnsemble ICE1712 (Envy24) (SND_ICE1712) [N/m/?] n
  ICE/VT1724/1720 (Envy24HT/PT) (SND_ICE1724) [N/m/?] n
  Intel/SiS/nVidia/AMD/ALi AC97 Controller (SND_INTEL8X0) [N/m/?] n
  Intel/SiS/nVidia/AMD MC97 Modem (SND_INTEL8X0M) [N/m/?] n
  Korg 1212 IO (SND_KORG1212) [N/m/?] n
  Digigram Lola (SND_LOLA) [N/m/?] n
  Digigram LX6464ES (SND_LX6464ES) [N/m/?] n
  ESS Allegro/Maestro3 (SND_MAESTRO3) [N/m/?] n
  Digigram miXart (SND_MIXART) [N/m/?] n
  NeoMagic NM256AV/ZX (SND_NM256) [N/m/?] n
  Digigram PCXHR (SND_PCXHR) [N/m/?] n
  Conexant Riptide (SND_RIPTIDE) [N/m/?] n
  RME Digi32, 32/8, 32 PRO (SND_RME32) [N/m/?] n
  RME Digi96, 96/8, 96/8 PRO (SND_RME96) [N/m/?] n
  RME Digi9652 (Hammerfall) (SND_RME9652) [N/m/?] n
  Studio Evolution SE6X (SND_SE6X) [N/m/?] (NEW) m
  S3 SonicVibes (SND_SONICVIBES) [N/m/?] n
  Trident 4D-Wave DX/NX; SiS 7018 (SND_TRIDENT) [N/m/?] n
  VIA 82C686A/B, 8233/8235 AC97 Controller (SND_VIA82XX) [N/m/?] n
  VIA 82C686A/B, 8233 based Modems (SND_VIA82XX_MODEM) [N/m/?] n
  Asus Virtuoso 66/100/200 (Xonar) (SND_VIRTUOSO) [N/m/?] n
  Digigram VX222 (SND_VX222) [N/m/?] n
  Yamaha YMF724/740/744/754 (SND_YMFPCI) [N/m/?] n
#
# configuration written to .config
#

```
Done! Now we have the minimum number of modules configured!

###Setting The Kernel Name [^quote2]
To give a different name of this minimum kernel image, we open the `.config` file under the root directory of the source code. And set the `CONFIG_LOCALVERSION` to `mini`, which will append `mini` to the kernel image name.

**In File .config:**
![configName]({{site.url}}/assets/img/postImgs/2017-02-10-ComplieTheLinuxKernel/configName.png)

Next, we just compile our kernel and install it as we did before:

```
$ sudo make -j2
$ sudo make modules install
$ sudo make install
```

### Restart And Enjoy The Mini Kernel
After restart, we get our Ubuntu16.04 running on a new minimal kernel! The size of the kernel image is only **21MB**.

![mini]({{site.url}}/assets/img/postImgs/2017-02-10-ComplieTheLinuxKernel/mini.png)

In the screen shot above, we find the currently using kernel is `Linux4.6.3-mini`, with size 21MB. And there are totally 3 kernels on this system. The original one from Ubuntu16.04, the 'big' 4.6.3 kernel and the 'mini' 4.6.3 kernel. 

##References
[^linux documentation]: Linux Kernel Documentation, https://www.kernel.org/doc/html/latest/

[^localmodconfig]: Linux Documentation. https://kernelnewbies.org/Linux_2_6_32#head-11f54cdac41ad6150ef817fd68597554d9d05a5f

[^quote]: Blog: Good and quick kernel configuration creation, Thorsten Leemhuis, 2012, http://www.h-online.com/open/features/Good-and-quick-kernel-configuration-creation-1403046.html

[^quote1]: Stackover, How to build a custom kernel with localmodconfig that support hardware of multiple machines, Pro Backup, http://unix.stackexchange.com/questions/119876/how-to-build-a-custom-kernel-with-localmodconfig-that-support-hardware-of-multip

[^quote2]: Stackover: how to change kernel version string, alk, 2012, http://stackoverflow.com/questions/12910969/buildroot-how-to-change-kernel-version-string
