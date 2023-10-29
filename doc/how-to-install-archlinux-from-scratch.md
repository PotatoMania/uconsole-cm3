# How to install ArchLinux on uConsole/CM3 from scratch

_DRAFT_

> It looks like there's a more detailed guide at [Autianic/clockworkpi-linux](https://github.com/Autianic/clockworkpi-linux#arch-linux-arm-chroot-environment-setup). It tells how to build the packages and how to make the partitions.
> The exception is that CM3 can only boot from a fat partition on a MBR partition table, with a proprietary bootloader. So no u-boot related stuff is necessary.

A Linux-based host computer(a VM is fine) is required for this guide. In best case, an ArchLinux host. If OS installation is not preferred/available, live CDs can be used.

This guide assumes that a ArchLinux host is used. Please make necessary changes when operating with other distros.

## Create partitions

Here we choose MBR partition scheme. If GPT(GUID partition Table) is preferred, please refer [make partitions gpt](./make_partitions_gpt.md).

Any tool can be used. The goals are:

- Create an empty MBR partition table
- Create a FAT32 primary partition
- Create EXT4 primary partition

If you've already done this, skip to next section.

Here fdisk is used(`/dev/sdx` is the SD card):

```
> sudo fdisk /dev/sdx
[sudo] password for hyx:

Welcome to fdisk (util-linux 2.39.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): o
Created a new DOS (MBR) disklabel with disk identifier 0xf12e4306.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-62357503, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-62357503, default 62357503): +512M

Created a new partition 1 of type 'Linux' and of size 512 MiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (2-4, default 2):
First sector (1050624-62357503, default 1050624):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (1050624-62357503, default 62357503):

Created a new partition 2 of type 'Linux' and of size 29.2 GiB.

Command (m for help): t
Partition number (1,2, default 2): 1
Hex code or alias (type L to list all): 0c

Changed type of partition 'Linux' to 'W95 FAT32 (LBA)'.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Now format the partitions:
```
mkfs.vfat /dev/sdx1
mkfs.ext4 /dev/sdx2
```

Simple & scripted version: TBW.

## Prepare the rootfs

__Please use root user in this section!__

### preparation

You need to install:

```
qemu-user-static
qemu-user-static-binfmt
arch-install-scripts
```

### steps

Mount the partitions:

```
mount /dev/sdx2 /mnt
mkdir /mnt/boot
mount /dev/sdx1 /mnt/boot
```

Download the latest tarball: http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz

Extract as the root user (not via sudo) using bsdtar to preserve extended attributes and ACLs:
```
bsdtar -xpf ArchLinuxARM-aarch64-latest.tar.gz -C /mnt
```

Generate the `fstab`:
```
genfstab -U /mnt >> /mnt/etc/fstab
```

_Note: inspect `/mnt/etc/fstab` to make sure no partitions on the host is included._

Copy the package `linux-uconsole-cm3-rpi64`(build it yourself with "/PKGBUILDs/linux-uconsole-cm3-rpi64") to the rootfs(/mnt)

Enter the rootfs with `arch-chroot` and install the packages:

```
arch-chroot /mnt
# initialize the keyring
pacman-key --init
pacman-key --populate archlinuxarm
# then install packages
pacman -Sy raspberrypi-bootloader firmware-raspberrypi
pacman -U --noconfirm linux-uconsole-cm3-rpi64*.pkg.zst
```

_Note: preinstalled `linux-aarch64` is uninstalled when installing `linux-uconsole-cm3-rpi64`._

Build and install the AUR package `brcmfmac43456-firmware` to have necessary WiFi firmware installed.

Now quit the rootfs.

Edit `/mnt/boot/config.txt` and write the following text:

```
ignore_lcd=1
disable_fw_kms_setup=1
max_framebuffers=2
arm_boost=1

# setup headphone detect pin
gpio=10=ip,np

# boot kernel directly
kernel=Image.gz
arm_64bit=1
initramfs initramfs-linux.img followkernel

# overlays
dtoverlay=dwc2,dr_mode=host
dtoverlay=vc4-kms-v3d
dtoverlay=audremap,pins_12_13
dtparam=audio=on
dtoverlay=uconsole
```

_Note: The kernel command line can be changed by editing `/mnt/boot/cmdline.txt`._

Unmount the partitions:

```
umount /mnt/boot /mnt
```

You are ready to boot this SD card on uConsole.
