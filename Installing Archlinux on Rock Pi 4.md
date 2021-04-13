Installing Archlinux on Rock Pi 4
===================================
**Does not work. This is just a documentation of my results, what I found out
and what might be a good starting point for someone else, trying the same
thing. Hope it helps.**

https://www.armbian.com/rock-pi-4/

Useful links
--------------
Working images for Rock Pi 4 which might be useful templates:
https://wiki.radxa.com/Rockpi4/install/microSD
https://wiki.radxa.com/Rockpi4/downloads

Guide for Rock64:
https://archlinuxarm.org/platforms/armv8/rockchip/rock64

Rockchip wiki:
http://opensource.rock-chips.com/wiki_Boot_option#Boot_from_SD.2FTF_Card

Someone else with similar problems:
https://medium.com/@AmadorUAVs/rock-pi-4-how-to-set-up-a-board-with-barely-any-official-documentation-33b73311ac01

Step by step installation
---------------------------
Zero the beginning of the SD card:

    dd if=/dev/zero of=/dev/sdX bs=1M count=32

Start parted to partition the SD card:

    parted /dev/sdX

At the parted prompt, create 5 new partitions:

  * Type `unit s` to use sectors as unit.
  * Type `mklabel gpt` to create the partition table.
  * Type `mkpart logical ext2 64 8063` to create the first partition. It will tell you that `The resulting partition is not properly aligned for best performance.`. `Ignore` this warning.
  * Type `mkpart logical ext2 16384 24575` to create the second partition.
  * Type `mkpart logical ext2 24576 32767` to create the third partition.
  * Type `mkpart logical fat32 32768 262143` to create the EFI boot partition.
  * Type `mkpart logical ext2 262144 100%` to create the linux filesystem partition.
  * Type `set 4 boot on` to set the filesystem type for partition 4 to EF00

As a result you should get the following partition table (this output is the
partition table of the official Ubuntu server image for Rock Pi 4):

    Disk /dev/sdh: 29.7 GiB, 31914983424 bytes, 62333952 sectors
    Disk model: STORAGE DEVICE  
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: gpt
    Disk identifier: 992DF2C2-0170-4F66-9492-FBF320673EEF
    
    Device      Start      End  Sectors  Size Type
    /dev/sdh1      64     8063     8000  3.9M Linux filesystem
    /dev/sdh2   16384    24575     8192    4M Linux filesystem
    /dev/sdh3   24576    32767     8192    4M Linux filesystem
    /dev/sdh4   32768   262143   229376  112M EFI System
    /dev/sdh5  262144 62333918 62071775 29.6G Linux filesystem

Create the filesystems for the EFI and the linux partition:

    mkfs.fat -F32 /dev/sdX4
    mkfs.ext4 /dev/sdX5

Write the data for the three other partitions:

    wget http://os.archlinuxarm.org/os/rockchip/boot/rock64/idbloader.img
    wget http://os.archlinuxarm.org/os/rockchip/boot/rock64/uboot.img
    wget http://os.archlinuxarm.org/os/rockchip/boot/rock64/trust.img
    dd if=idbloader.img of=/dev/sdX seek=64
    dd if=uboot.img of=/dev/sdX seek=16384
    dd if=trust.img of=/dev/sdX seek=24576

Mount the partitions:

    mkdir root
    mount /dev/sdX5 root
    mkdir boot
    mount /dev/sdX4 boot

Download and install the ArchLinuxARM base:

    wget http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz
    bsdtar -xpf ArchLinuxARM-aarch64-latest.tar.gz -C root

Move the boot data to the boot partition and register this new partition in
the `fstab`:

    mv root/boot/* boot/
    echo "/dev/mmcblk0p4 /boot vfat defaults 0 1" >> root/etc/fstab

Ensure that everything is written to the SD card:

    sync

This is how far I got. It does not boot. I guess, one issue is, that the boot
partition does not really contain the EFI data. But I also guess, that there
are more issues.

If you get further than me, update this guide and let me now!