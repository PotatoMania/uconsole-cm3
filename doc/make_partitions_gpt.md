# Make partitions with GPT and hybrid MBR

- GPT: GUID Partition Table
- MBR: Master Boot Record, take it as an old-fashioned partition table

Raspberry Pi 3 only support boot from a FAT32 partition with MBR scheme. GPT has provided a hybrid setup to support MBR systems. For RPi 3, you only need to make the boot partition(FAT32, containing the `bootcode.bin`) visible on the hybrid MBR.

Here's an example creating such setup on a 64G micro SD card using `gdisk`:

```
root # gdisk /dev/sdd
GPT fdisk (gdisk) version 1.0.9.1

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): o
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): Y

Command (? for help): n
Partition number (1-128, default 1):
First sector (34-125173726, default = 2048) or {+-}size{KMGTP}: 32M
Last sector (65536-125173726, default = 125171711) or {+-}size{KMGTP}: +480M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): ef00
Changed type of partition to 'EFI system partition'

Command (? for help): n
Partition number (2-128, default 2):
First sector (34-125173726, default = 1048576) or {+-}size{KMGTP}: +40G
Last sector (84934656-125173726, default = 125171711) or {+-}size{KMGTP}: ^C
hyx@ISC-CAT5 ~> sudo gdisk /dev/sdd                                         130
GPT fdisk (gdisk) version 1.0.9.1

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): o
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): Y

Command (? for help): n
Partition number (1-128, default 1):
First sector (34-125173726, default = 2048) or {+-}size{KMGTP}: 32M
Last sector (65536-125173726, default = 125171711) or {+-}size{KMGTP}: +480M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): ef00
Changed type of partition to 'EFI system partition'

Command (? for help): n
Partition number (2-128, default 2):
First sector (34-125173726, default = 1048576) or {+-}size{KMGTP}:
Last sector (1048576-125173726, default = 125171711) or {+-}size{KMGTP}: +50G
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300):
Changed type of partition to 'Linux filesystem'

Command (? for help): r

Recovery/transformation command (? for help): h

WARNING! Hybrid MBRs are flaky and dangerous! If you decide not to use one,
just hit the Enter key at the below prompt and your MBR partition table will
be untouched.

Type from one to three GPT partition numbers, separated by spaces, to be
added to the hybrid MBR, in sequence: 1
Place EFI GPT (0xEE) partition first in MBR (good for GRUB)? (Y/N): N

Creating entry for GPT partition #1 (MBR partition #1)
Enter an MBR hex code (default EF): C
Set the bootable flag? (Y/N): y

Unused partition space(s) found. Use one to protect more partitions? (Y/N): N

Recovery/transformation command (? for help): o

Disk size is 125173760 sectors (59.7 GiB)
MBR disk identifier: 0x00000000
MBR partitions:

Number  Boot  Start Sector   End Sector   Status      Code
   1      *          65536      1048575   primary     0x0C
   2                     1        65535   primary     0xEE

Recovery/transformation command (? for help): p
Disk /dev/sdd: 125173760 sectors, 59.7 GiB
Model: uSD UHS2 RDR
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 2BFA76D5-5A3B-423F-A92D-F7D2CF418092
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 125173726
Partitions will be aligned on 2048-sector boundaries
Total free space is 19333053 sectors (9.2 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1           65536         1048575   480.0 MiB   EF00  EFI system partition
   2         1048576       105906175   50.0 GiB    8300  Linux filesystem

Recovery/transformation command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdd.
The operation has completed successfully.
```

In summary:

- create a new GPT partition table
- create the partitions
- create the hybrd MBR
- write everything

The above example is adapted from [Gentoo Wiki](https://wiki.gentoo.org/wiki/Hybrid_partition_table).

Please adapt the partitions sizes accordingly.

Now you can format your partitions with `mkfs.vfat` and `mkfs.ext4`.
