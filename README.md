# Cross-Compiling the Kernel

First, you will need a suitable Linux cross-compilation host. We tend to use Ubuntu; since Raspberry Pi OS is also a Debian distribution, it means many aspects are similar, such as the command lines.

You can either do this using VirtualBox (or VMWare) on Windows, or install it directly onto your computer. For reference, you can follow instructions online at [Wikihow](https://www.wikihow.com/Install-Ubuntu-on-VirtualBox).

### Install Required Dependencies and Toolchain for 32bit Kernel

To build the sources for cross-compilation, make sure you have the dependencies needed on your machine by executing

> sudo apt install git bc bison flex libssl-dev make libc6-dev libncurses5-dev

If you find you need other things, please submit a pull request to change the documentation.

> sudo apt install crossbuild-essential-armhf

### Get the Kernel Sources

To download the minimal source tree for the current branch, run:

> git clone https://github.com/open-astro/linux.git -b rpi-5.10.AAP

### Build sources

Enter the following commands to build the sources and Device Tree files:

For Raspberry Pi Compute Module 4:

> cd linux
> KERNEL=kernel7l
> make ARCH=arm CROSS\_COMPILE=arm-linux-gnueabihf- bcm2711\_defconfig

*NOTE*
To speed up compilation on multiprocessor systems, and get some improvement on single processor ones, use *-j n*, where n is the number of processors \* 1.5. You can use the *nproc* command to see how many processors you have. Alternatively, feel free to experiment and see what works!

> make ARCH=arm CROSS\_COMPILE=arm-linux-gnueabihf- zImage modules dtbs

### Install Directly onto the SD Card

Having built the kernel, you need to copy it onto your Raspberry Pi and install the modules; this is best done directly using an SD card reader.

First, plug in you Rasberry Pi Compute 4 into the Rasberry Pi Compute Module 4 IO Boarard using the jumper and the Micro USB Cable and connect to the board uisng the following command

> sudo ./rpiboot

use *lsblk* before and after and identify the board. You should end up with something a lot like this:

> sdb
> 
> 
> > sdb1
> > sdb2

with sdb1 being the FAT filesystem (boot) partition, and sdb2 being the ext4 filesystem (root) partition.

Mount these first, adjusting the partition letter as necessary:

> mkdir mnt
> mkdir mnt/fat32
> mkdir mnt/ext4
> sudo mount /dev/sdb1 mnt/fat32
> sudo mount /dev/sdb2 mnt/ext4

*NOTE*
You should adjust the drive letter appropriately for your setup, e.g. if your SD card appears as /dev/sdc instead of /dev/sdb.

Next, install the kernel modules onto the SD card:

> sudo env PATH=$PATH make ARCH=arm CROSS\_COMPILE=arm-linux-gnueabihf- INSTALL\_MOD\_PATH=mnt/ext4 modules\_install

> sudo cp mnt/fat32/$KERNEL.img mnt/fat32/$KERNEL-backup.img
> sudo cp arch/arm/boot/zImage mnt/fat32/$KERNEL.img
> sudo cp arch/arm/boot/dts/*.dtb mnt/fat32/*
> *sudo cp arch/arm/boot/dts/overlays/*.dtb\* mnt/fat32/overlays/
> sudo cp arch/arm/boot/dts/overlays/README mnt/fat32/overlays/
> sudo cp renesas\_usb\_fw.mem mnt/ext4/lib/firmware/
> sudo umount mnt/fat32
> sudo umount mnt/ext4

At this point you have copied over the new kernel which supports renesas\_usb and also the copied over the module needed for the renesas\_usb file and can now boot the RPi Compute Module 4 in the ASIAIR Plus.
