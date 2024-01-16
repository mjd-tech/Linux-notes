# Partitions and Formatting
A storage device (Hard disk, SSD, USB stick, SD-Card, etc) needs at least one "partition", which is "formatted" with a "file system".
The "partition table" defines where the partitions are located on the device. 
This information is stored in a separate area of the device, apart from the partition(s)

## Partition table
There are 2 main types in use:

1.  the classic MBR (Master Boot Record)
2.  the modern GPT (GUID Partition Table)

## MBR (Master Boot Record)

- MBR supports up to 4 "primary" partitions.
- If you need more partitions, you create an "extended partition".
- An extended partition is a "container" for "logical partitions".
- The traditional hard disk block size of 512 bytes limits MBR partitions to 2TB in size.

Windows must be installed to a primary partition.

In Linux, the customary practice is to create 3 primary partitions: sda1 sda2 sda3, followed by 1 extended partition sda4.  
The logical partitions on sda4 are numbered sda5, sda6, etc.

- MBR resides in the first physical sector of the disk
- 512 bytes in size
- First 446 bytes are the bootstrap code (bytes 0-445 or hex 0000-01BD)
- Next is the partition table (4 entries of 16 bytes each) total 64 bytes
- Lastly, a 2 byte boot signature that is supposed to uniquely identify the disk.

### MBR Partition Table

- Begins at offset 446 (hex 01BE)
- Contains 4 16-byte entries.

| Bytes | Description                    | Notes                                                           |
| :---- | :----------------------------- | :-------------------------------------------------------------- |
| 1     | Bootable flag                  | hex 80 (bootable) or 00 (non-bootable)                          |
| 2-4   | Start of partition             | CHS (cylinder, head, sector) format                             |
| 5     | Partition Type                 | 07=NTFS 0C=FAT32 82=SWAP 83=LINUX FD=LINUX-RAID, etc            |
| 6-8   | End of partition               | CHS format                                                      |
| 9-12  | Start of partition             | LBA (Logical Block Address) format.                             |
| 13-16 | Number of sectors in partition | The size of the partition, not the last sector of the partition |

- The CHS format is an artifact from the 1980s IBM PC/XT
- CHS can be converted to LBA, but it depends on the "disk geometry", there is not just one formula.
- This stuff is done in BIOS and on the drive itself, so you rarely need to worry about CHS
- The last LBA sector of the partition can be derived from Start LBA + Size

## GUID Partition Table

- GPT does not have primary and logical partitions, just *partitions*
- The number of partitions is theoretically unlimited, most implementations limit to 128 partitions.
- Partitions can be up to 18 exabytes in size.
- GPT keeps copies of the partition table in the first and last sector on the disk
- The first sector (LBA 0) of the disk is reserved for a "protective MBR" 
  such that booting a BIOS-based computer from a GPT disk is supported,
  but the boot loader and operating system must both be GPT-aware.
- The GPT header begins on the second sector (LBA 1). Typically, the next 32 sectors
  are used for partition tables and sector 34 is the first usable sector on the disk.
- In practice, the first partition will typically begin at LBA 2048 (1Mbyte boundary).

## Choosing between GPT and MBR

- It is recommended to use always GPT for UEFI boot as some UEFI firmwares do not allow UEFI-MBR boot.
- If using GRUB-legacy as the bootloader, you must use MBR.
- To dual-boot with Windows (both 32-bit and 64-bit) using Legacy BIOS, you must use MBR.
- To dual-boot Windows 64-bit using UEFI instead of BIOS, you must use GPT.
- Choose MBR on older hardware, because its BIOS might not support GPT.
- If none of the above apply, choose freely between GPT and MBR; since GPT is more modern, it is recommended in this case.

## Linux Partition schemes

| Scheme                             | Description                                                                                           |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Single root / partition            | This is the simplest. A **swapfile** can be created and easily resized as needed                      |
| Single root /, with swap partition | A swap partition has (slightly) better performance than a swap file                                   |
| Root /, swap, /home                | Makes it easier to upgrade or change the operating system without losing user data or settings.       |
| separate /boot partition           | stores kernel and bootloader stuff. Often is ext2 (non-journaling) for (slightly) better performance. |
| separate /var partition            | useful for servers such as Web servers, as the websites and MySQL databases are kept in /var.         |

### How big should my partitions be?

The size of the partitions depends on personal preference, but the
following information may be helpful:

| Part. | Size     | Notes                                                                          |
| :---- | :------- | :----------------------------------------------------------------------------- |
| /boot | 500 MB   | It requires only about 100 MB, but you want more space for multiple kernels    |
| /     | 15-20 GB | If you plan to store a swap file here, you might need a larger partition size. |
| /var  | varies   | 8-12 GB for a desktop system. Much larger for a server.                        |
| /home | varies   | On a desktop system, /home is typically the largest filesystem                 |
| swap  | varies   | See below                                                                      |
| /data | varies   | Optional partition for files to be shared by all users.                        |

RedHat swap space recommendations:

    Amount of RAM in the system   Recommended swap space         Recommended swap space 
                                                                 if allowing for hibernation
    ---------------------------   ----------------------------   ---------------------------
    2GB of RAM or less            2 times the amount of RAM      3 times the amount of RAM
    2GB to 8GB of RAM             Equal to the amount of RAM     2 times the amount of RAM
    8GB to 64GB of RAM            0.5 times the amount of RAM    1.5 times the amount of RAM
    64GB of RAM or more           4GB of swap space              No extra space needed

## Partitioning tools

- gparted — Graphical tool. Handles MBR, GPT, formatting, labelling, moving, resizing. Try to use this if at all possible!
- parted — Terminal version of gparted.
- fdisk — Terminal partitioning tool. Supports GPT since version 2.23 (2015)
- cfdisk — Terminal partitioning tool written with ncurses libraries.  Supports GPT since version 2.25 (2015)
- gdisk — GPT version of fdisk. Because fdisk did not support GTP until 2015
- cgdisk — GPT version of cfdisk. Because cfdisk did not support GTP until 2015

Note: fdisk,cfdisk, and parted are installed by default on most Linux distros.
gdisk and cgdisk might not be installed, since modern fdisk and cfdisk can handle GPT.

## Partition alignment

Way back in the MSDOS days, a BIOS interacted with drives by knowing the exact geometry of the drive,
namely how many cylinders, heads, and sectors were on a disk (**CHS** Addressing).
Disks had physical sectors of 512 bytes. This became a standard.

When newer, larger drives appeared, the number of sectors per track varied 
(tracks on the outside of the platter can hold more sectors than those on the inside), 
this broke CHS.

So **LBA** (Logical Block Address) was born. LBA holds the number of sectors per track constant at 63, 
and varies the number of heads and cylinders, which works for the old-school BIOSes.

The first partition always began at sector 64 (LBA 63. LBA starts at zero). 
This maintained compatibility with MSDOS, BIOS and the partitioning tools of the day.

Unfortuantely the decision to set sectors per track to 63 later became a problem. 
The number 63 is not a power of 2.

Newer hard drives have a physical sector size of 4,096 bytes, 
and emulate a "logical sector size" of 512 bytes to maintain backward compatibility.
File systems such as ext4 and Windows NTFS use block sizes of 4096. 
So even a one-byte file will consume 4096 bytes, one physical sector, 8 logical sectors. 
So the idea is to get the physical sectors to align with the file system's block size for best performance.

If the first partition starts at LBA 63, it straddles 2 4096B physical
sectors. This means every block that gets written needs to write 2
physical sectors and this harms performance. The problem is even worse
with SSD drives some of which use 128kB "sectors".

To avoid these problems, alignment at one-megabyte boundaries is
recommended. That is, 2048 logical 512 byte sectors.

Newer version of Windows (Windows Vista, Windows 7 and Windows Server 2008) 
perform the following, reasonable alignment:

- Disk sizes less than or equal to four gigabytes should be aligned on
  sixty-four-kilobyte boundaries
- Disk sizes larger than four gigabytes should be aligned on
  one-megabyte boundaries

In Linux, older version of fdisk prior to verion 2.17.1 ( Feb. 2010)
defaulted to msdos mode. Misalignment will occur using fdisk, if special
parameters are not used. The newer fdisk from Version 2.17.1 will align
on the one megabyte boundary by default.

### Solid state drives

Solid state drives are based on flash memory, and thus differ
significantly from hard drives. While reading remains possible in a
random access fashion, erasure (hence rewriting and random writing) is
possible only by whole blocks. Additionally, the erase block size (EBS)
are significantly greater than regular block size, for example 128KiB
vs. 4KiB, so it is necessary to align to multiples of EBS.

To verify a partition is aligned, query it using *blockdev* as shown
below, if a '0' is returned, the partition is aligned:

    sudo blockdev --getalignoff /dev/<partition>
    0

## Formatting

A partion needs to be formatted. The exception is the swap partition.  
In linux the standard terminal tool is *mkfs*. In actuality, mkfs is
simply a front-end for the various filesystem builders (mkfs.fstype)
available under Linux. For example:

    # need to be root, so add sudo
    sudo mkfs -t ext4 /dev/sda2
    # is the same as
    sudo mkfs.ext4 /dev/sda2

To see what filesystems you can create on your system:

```
apropos mkfs

jfs_mkfs (8)         - create a JFS formatted partition
mkfs (8)             - build a Linux filesystem
mkfs.bfs (8)         - make an SCO bfs filesystem
mkfs.btrfs (8)       - create a btrfs filesystem
mkfs.cramfs (8)      - make compressed ROM file system
mkfs.exfat (8)       - create an exFAT filesystem
mkfs.ext2 (8)        - create an ext2/ext3/ext4 file system
mkfs.ext3 (8)        - create an ext2/ext3/ext4 file system
mkfs.ext4 (8)        - create an ext2/ext3/ext4 file system
mkfs.f2fs (8)        - create an F2FS file system
mkfs.fat (8)         - create an MS-DOS FAT filesystem
mkfs.jfs (8)         - create a JFS formatted partition
mkfs.minix (8)       - make a Minix filesystem
mkfs.msdos (8)       - create an MS-DOS FAT filesystem
mkfs.ntfs (8)        - create an NTFS file system
mkfs.reiserfs (8)    - The create tool for the Linux ReiserFS filesystem.
mkfs.vfat (8)        - create an MS-DOS FAT filesystem
mkfs.xfs (8)         - construct an XFS filesystem
```

## Partition Label
You might want to give a partition a name. It will have a UUID, but
sometimes the name is easier to deal with. Especially with removable
media. For ext2,3,4

    # show label
    e2label /dev/device
    # Set label
    e2label /dev/device new-label-name-here

labels are limited to 16 characters. Save yourself some grief and stick
to a-Z 0-9 \_ (underscore) and don't use spaces.

## Backup / Restore Partition Table

Modern (post 2015) cfdisk supports both MBR and GPT partition tables:

    Backup
    # sfdisk -d /dev/sda > /path/to/sda_ptable

    Restore
    # sfdisk /dev/sda < /path/to/sda_ptable

    Replicate /dev/sda to dev/sdd (for RAID array)
    # sfdisk -d /dev/sda | sfdisk -f /dev/sdd
