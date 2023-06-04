# uLinux - A small somewhat functional Linux OS (UNFINISHED)
I had this on a text file somewhere on my computer for a while and decided now to make it public. *Thank you Prof. David Freitas for the sugestion*

I will try to make this as understandable as possible, explaining how stuff works.

**DISCLAIMER:** I am not an expert on Linux or even Operating Systems.

## Index
Try to not get lost 🙂
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
   - [Including Programs](#including-programs)
   - [Creating the Image](#creating-the-image)
 - [Making the ISO](#making-the-iso)
   - [Bootloader](#bootloader)
   - [Image's Structure](#images-structure)
   - [The ISO File](#the-iso-file)

## Intro
After learning that a Linux distro is just the Kernel packed with diferent utilites, I started wondering if I could make my own.
The first place I landed was [Linux From Scratch](https://www.linuxfromscratch.org/). But after 1 or 2 hours of running `make`, `make check`/`make test` and `make install` I got bored and wanted to find a way to do this without building so many packages.

I would still recommend trying LFS, but keep in mind you will need to dedicate a weekend to the system.

The idea here is booting the Linux Kernel and using [BusyBox](https://www.busybox.net) to provide some of the most common utilities for the system to be somewhat usable (Don't expect to run this as your personal OS).
For the bootloader I chose syslinux because of how small and easy it is to setup.

**This assumes you are already using Linux or GNU/Linux as a OS**

**Before everything else, create a folder for the project, maybe something like uLinux.**

**You will need a C/C++ compiler and xorriso. To install these, Google (or DuckDuckGo) is your friend**

## Linux Kernel
Every respectable Linux Distros (and even the most idiotic ones) include the Linux Kernel. Well, our's is no different.

### Finding a Kernel Version
All(??) kernel versions can be found in [kernel.org](https://kernel.org/pub/linux/kernel). The kernel sources are usually called something like `linux-{the version}.tar.gz` or `.tar.xz`.
I went with the latest version 5 kernel (5.19.9) but any recent version should work (*Note: Not all versions were tested, some might not work*)

After selecting a kernel version, download the tarball and extract it inside the project's folder (`tar -xf linux-{version}.tar.?z`). A new folder called *linux-{version}* should now exist. 

### Building the Kernel
Go inside the extracted folder.

In here run `make defconfig`. This should configure the kernel with the default options for your architecture. (To modify any of the settings, run `make menuconfig`)

Now run ```make -j`nproc` ``` to build the kernel. This might take a while
 - **-j** - Number of jobs. How many files are being compiled at the same time
 - **`nproc`** - Get's the output of nproc. Nproc outputs how many cpu cores/threads your system has

In the end you should see a message telling where the build kernel was placed. (For x86: arch/x86/boot). Copy the bzImage file to the projet's folder and rename it to vmlinuz

## BusyBox

### Finding a BusyBox Version

### Building BusyBox


## InitRD

### Creating Folders

### Including Programs

### Creating the Image


## Making the ISO

### Bootloader

### Image's Structure

### The ISO file