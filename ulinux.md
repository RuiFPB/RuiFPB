# uLinux - A small somewhat functional Linux OS
I had this on a text file somewhere on my computer for a while and decided now to make it public. 

*Thank you, Prof. David Freitas, for the sugestion*

----

I will try to make this as understandable as possible, explaining how stuff works.

**DISCLAIMER:** I am not an expert on Linux or even Operating Systems.

## Index
<details>
 <summary>Try to not get lost 🙂</summary>

 - [Index](#index)
 - [Intro](#intro)
 - [Linux Kernel](#linux-kernel)
   - [Finding a Kernel Version](#finding-a-kernel-version)
   - [Building The Kernel](#building-the-kernel)
 - [BusyBox](#busybox)
   - [Finding a BusyBox Version](#finding-a-busybox-version)
   - [Building BusyBox](#building-busybox)
 - [initrd.img](#initrd)
   - [Creating the Folders](#creating-folders)
   - [Creating the Image](#creating-the-image)
 - [Making the ISO](#making-the-iso)
   - [Bootloader](#bootloader)
   - [Image's Structure](#images-structure)
   - [The ISO File](#the-iso-file)
 - [Debugging](#debugging)
</details>

## Intro
After learning that a Linux distro is just the Kernel packed with diferent utilites, I started wondering if I could make my own.
The first place I landed was [Linux From Scratch](https://www.linuxfromscratch.org/). But after 1 or 2 hours of running `make`, `make check`/`make test` and `make install` I got bored and wanted to find a way to do this without building so many packages.

I would still recommend trying LFS, but keep in mind you will need to dedicate a weekend to the system.

The idea here is booting the Linux Kernel and using [BusyBox](https://www.busybox.net) to provide some of the most common utilities for the system to be somewhat usable (Don't expect to run this as your personal OS).
For the bootloader I chose syslinux because of how small and easy it is to setup.

**This assumes you are already using Linux or GNU/Linux as a OS**

**Before everything else, create a folder for the project, maybe something like uLinux.**

**You will need a C/C++ (gcc recommended) compiler and xorriso. To install these, Google (or DuckDuckGo) is your friend**

At the end you should have the follwing structure inside your project's folder:

 - initrd/
   - bin/
     - busybox
   - dev/
   - proc/
   - sys/
   - init
 - cdboot/
   - vmlinuz
   - initrd.img
   - isolinux/
     - isolinux.bin
     - ldlinux.c32
     - isolinux.cfg
  - linux-{version}/...
  - busybox-{version}/...
  - syslinux-{version}/...
  - initrd.img
  - vmlinuz
  - ulinux.iso

## Linux Kernel
Every respectable Linux Distro (and even the most idiotic ones) include the Linux Kernel. Well, our's is no different.

### Finding a Kernel Version
All(need to check that) kernel versions can be found in [kernel.org/pub/linux/kernel](https://kernel.org/pub/linux/kernel). The kernel sources are usually called something like *linux-{the version}.tar.gz* or *.tar.xz*.
I went with the latest version 5 kernel (5.19.9) but any recent version should work.

*Note: Not all versions were tested **obviously***

After selecting a kernel version, download the tarball and extract it inside the project's folder (`tar -xf linux-{version}.tar.xz`). A new folder called *linux-{version}* should now exist. 

### Building the Kernel
Go inside the extracted folder.

In here run `make defconfig`. This should configure the kernel with the default options for your architecture. (To modify any of the settings, run `make menuconfig`)

Now run ```make -j`nproc` ``` to build the kernel. This might take a while
 - **-j** - Number of jobs. How many files are being compiled at the same time
 - **\`nproc\`** - Outputs how many cpu cores/threads your system has

In the end you should see a message telling where the build kernel was placed. (For x86: arch/x86/boot). Copy the **bzImage** file to the projet's folder and rename it to **vmlinuz**

## BusyBox
As I stated in the intro, we will use BusyBox to provide the expected utilities to make the system work enough for us to test with it.

### Finding a BusyBox Version
The diferent busybox versions can be found in [busybox.net/downloads](https://www.busybox.net/downloads/). The sources are usually called *busybox-{version}.tar.bz*. I used 1.36.0 when testing.

Download the tarball, like we did for the kernel. *(**WARNING**: For every tarbal there is a .sha256 file with the hash for the download. Don't mistake these for the tarbal)*. Extract the files, and now a folder with busybox in the name should exist.

### Building BusyBox
Go inside the busybox sourcces folder and run `make defconfig`. From my testing changing a few options can avoid some bugs and errors, so run `make menuconfig` to change these.

On the menu, select Settings and search for *exec prefers applets* and *Build static binary (no shared libs)*. These should be enabled.

After that, go back and select Shells. In here find and enable *Standalone shell*

If you want, change any other options (ensure you know what they do). Finally, exit the menu saving your changes. 

To build, simply run `make`. Now you should have a executable called **busybox**. Copy it to the project's folder.

## InitRD
Now we need to create a initial filesystem for the kernel to use.

### Creating Folders
Inside the project's folder, create a folder called **initrd**.
Inside it create four more folders: 
 - bin
 - dev
 - proc
 - sys

*If you want to know the purpose of the folders, I recommend watching this quick video by Fireship about the Linux Directories: [Linux Directories Explained in 100 Seconds](https://www.youtube.com/watch?v=42iQKuQodW4)*

### Including BusyBox
To include BusyBox with the system, copy the executable compiled previously to the **bin** folder.

### Init script
Inside the initrd folder create a file with the following content:
```sh
#!/bin/busybox sh
/bin/busybox --install -s /bin
mount -t sysfs sysfs /sys
mount -t proc proc /proc
mount -t devtmpfs udev /dev

#change the kernel print level so we are not spammed after bootup with messages
sysctl -w kernel.printk="2 4 1 7"

#clear
exec /bin/sh
```

About the script:
 - `/bin/busybox --install -s /bin` - asks busybox to create links with the name of the actual applets/utilities in the bin folder. Ex.: a symlink called *vi* pointing to *busybox* to launch the vi editor. (Should not be needed if *exec prefers applets* was enabled, but we do it anyway)
 - The mount commands - Because the folders sys, proc and dev are populated by the kernel they use special filesystems so that the kernel can change the contents. Every distro does this, run `mount` on your pc and you will probably find these.
 - `sysctl -w kernel.printk="2 4 1 7"` - change the logging level to 2 (Only emergencies, alerts and critical messages) ([more info](https://www.kernel.org/doc/html/latest/core-api/printk-basics.html))
 - `exec /bin/sh` - runs the shell on startup. The shell will be process 1, so if you run `exit` or crash the shell, the kernel will panic.

Now to ensure the kernel can execute the script do `chmod 777 init`. This will enable read, write and execute permissions to everyone. (**Don't run this on your working linux system**)

### Creating the Image
Now we need to transform this folder into a .img file. To do that, sill inside the initrd folder run `find . | cpio -o -H newc > ../initrd.img`. This will create a initrd.img file on the project's folder by asking cpio to copy all files found with find to a new archive called initrd.img.

## Making the ISO
If you have virt-manager you can already boot the system by creating a VM and selecting the kernel and initrd.img.
Because some might want to run this on VirtualBox or other virtualization software we will create a ISO.

I will not explain every step. If you want to find out how something works, your search engine of choice is your best friend.

### Bootloader
We need a bootloader that the bios can recognize to boot the kernel. For that we will use syslinux
Download the syslinux tarball from [kernel.org/pub/linux/utils/boot/syslinux](https://kernel.org/pub/linux/utils/boot/syslinux/). I used version 6.03.

Save the tarball inside the project's folder and extract it

From the syslinux folder copy to the projet's folder the files **(syslinux-{version})/bios/core/isolinux.bin** and **(syslinux-{version})/bios/com32/elflink/ldlinux/ldlinux.c32**.


### Image's Structure
Now create a folder called cdboot on the project's folder. This will hold the files to later be used for our iso image.

Inside cdboot create a folder called isolinux and copy the isolinux.bin and ldlinux.c32 files from before to there.

Inside the isolinux folder create a new file called isolinux.cfg with the following:
```cfg
DEFAULT linux

LABEL linux
    KERNEL /vmlinuz
    APPEND initrd=/initrd.img
```

Inside cdboot place the vmlinuz and initrd.img files.

### The ISO file
Still with me? Now, from the project's folder run the following:
`xorriso -as mkisofs -o ulinux.iso -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table cdboot/`

This will use the xorriso tool to create a bootable CD image we can use.

## Debugging
I will not provide help with debugging.
The steps I usually do when something doesn't work are the following:
 - Verify my work. Rerun some steps if needed
 - Experiment with changing some script or command if I think that is causing the error.
 - Search online
