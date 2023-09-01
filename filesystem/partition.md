# Partitions and Formatting

## Partition table

Partition information is stored in the partition table; today, there are
2 main formats in use:

1.  the classic MBR (Master Boot Record)
2.  the modern GPT (GUID Partition Table)

The latter is an improved version that does away with several
limitations of MBR style.

## Master Boot Record

MBR originally only supported up to 4 partitions. Later on, extended and
logical partitions were introduced to get around this limitation. The
traditional hard disk block size of 512 bytes limits partitions to 2TB
in size.

**Types of partitions**

1.  Primary
2.  Extended
3.  Logical

**Primary partitions**

- can be bootable
- limited to 4 partitions per disk or RAID volume.
- A system that dual boots with Windows will require that Windows reside
  in a primary partition.
- If you need more than four partitions, an extended partition
  containing logical partitions is used.

**Extended partitions**

- A Container for logical partitions.
- Does not get formatted with a file system.
- A disk can contain only *one* extended partition.
- Counts as one of the 4 total primary partitions. You can have 3
  primaries and 1 extended.

**Logical Partitions**

- Require one extended partition to be set up. All logical partition fit
  inside the extended partition.
- get formatted with a file system.
- theoretically unlimited number of logical partitions. In practice,
  tools such a fdisk will only display about 60 total partitions.
- Windows cannot boot from a logical partition

In Linux, the customary practice is to create 3 primary partitions sda1
through sda3 followed by 1 extended partition sda4.  
The logical partitions on sda4 are numbered sda5, sda6, etc.

**Size**

- MBR resides in the first physical sector of the disk
- 512 bytes in size
- First 446 bytes are the bootstrap code (bytes 0-445 or hex 0000-01BD)
- Next is the partition table (4 entries of 16 bytes each) total 64
  bytes
- Lastly, a 2 byte boot signature that is supposed to uniquely identify
  the disk.

**Partition Table**

- Begins at offset 446 (hex 01BE)
- Contains 4 16-byte entries.
- For each entry:

| Bytes | Description                    | Notes                                                           |
|:------|:-------------------------------|:----------------------------------------------------------------|
| 1     | Bootable flag                  | hex 80 (bootable) or 00 (non-bootable)                          |
| 2-4   | Start of partition             | CHS (cylinder, head, sector) format                             |
| 5     | Partition Type                 | 07=NTFS 0C=FAT32 82=SWAP 83=LINUX FD=LINUX-RAID, etc            |
| 6-8   | End of partition               | CHS format                                                      |
| 9-12  | Start of partition             | LBA (Logical Block Address) format.                             |
| 13-16 | Number of sectors in partition | The size of the partition, not the last sector of the partition |

- The CHS format is an artifact from the 1980s IBM PC/XT
- CHS can be converted to LBA, but it depends on the "disk geometry",
  there is not just one formula.
- This stuff is done in BIOS and on the drive itself, so you rarely need
  to worry about CHS
- The last LBA sector of the partition can be derived from Start LBA +
  Size

## GUID Partition Table

**Types of Partitions**

- GPT does not have primary and logical partitions, just *partitions*

**Size and Number of Partitions**

- The amount of partitions per disk or RAID volume is theoretically
  unlimited, most implementations limit to 128 partitions.
- Partitions can be up to 18 exabytes in size.
- GPT has fault-tolerance by keeping copies of the partition table in
  the first and last sector on the disk
- The first sector (LBA 0) of the disk is reserved for a "protective
  MBR" such that booting a BIOS-based computer from a GPT disk is
  supported, but the boot loader and operating system must both be
  GPT-aware.
- The GPT header begins on the second sector (LBA 1). Typically, the
  next 32 sectors are used for partition tables and sector 34 is the
  first usable sector on the disk.
- In practice, the first partition will typically begin at LBA 2048
  (1Mbyte boundary).

## Choosing between GPT and MBR

- If using GRUB-legacy as the bootloader, one must use MBR.
- To dual-boot with Windows (both 32-bit and 64-bit) using Legacy BIOS,
  one must use MBR.
- To dual-boot Windows 64-bit using UEFI instead of BIOS, one must use
  GPT.
- If you are installing on older hardware, especially laptop, consider
  choosing MBR because its BIOS might not support GPT.
- If none of the above apply, choose freely between GPT and MBR; since
  GPT is more modern, it is recommended in this case.
- It is recommended to use always GPT for UEFI boot as some UEFI
  firmwares do not allow UEFI-MBR boot.

## Linux Partition schemes

|                                    |                                                                                                                                                  |
|------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| Single root / partition            | This is the simplest. A swapfile can be created and easily resized as needed                                                                     |
| Single root /, with swap partition | the default for many Linux distributions. A swap partition has (slightly) better performance than a swap file                                    |
| Root /, swap, /home                | Good choice for desktop systems. Makes it easier to upgrade or even change the operating system without losing user data or settings.            |
| separate /boot partition           | stores kernel and bootloader stuff. Often is ext2 (non-journalling) for (slightly) better performance. This stuff is mostly read, rarely written |
| separate /var partition            | useful for servers such as Web servers, as the websites and MySQL databases are kept in /var.                                                    |

### How big should my partitions be?

The size of the partitions depends on personal preference, but the
following information may be helpful:

|          |            |                                                                                                                                                            |
|:---------|:-----------|:------------------------------------------------------------------------------------------|
| /boot    | 500 MB     | It requires only about 100 MB, but you want more space for                                |
| /        | 15-20 GB   | If you plan to store a swap file here, you might need a larger partition size.                                                                             |
| /var     | varies     | 8-12 GB for a desktop system. Much larger for a server.                                                                                                    |
| /home    | varies     | On a desktop system, /home is typically the largest filesystem                            |
| swap     | varies     | See below                                                                                                                                                  |
| /data    | varies     | Optional partition for files to be shared by all users.  |

RedHat swap file recommendations:

    Amount of RAM in the system   Recommended swap space         Recommended swap space 
                                                                 if allowing for hibernation
    ---------------------------   ----------------------------   ---------------------------
    2GB of RAM or less            2 times the amount of RAM      3 times the amount of RAM
    2GB to 8GB of RAM             Equal to the amount of RAM     2 times the amount of RAM
    8GB to 64GB of RAM            0.5 times the amount of RAM    1.5 times the amount of RAM
    64GB of RAM or more           4GB of swap space              No extra space needed

## Partitioning tools

- gparted — Graphical tool. Handles MBR, GPT, formatting, labelling,
  moving, resizing. Try to use this if at all possible!
- parted — Terminal version of gparted.
- fdisk — Terminal partitioning tool. MBR only.
- cfdisk — Terminal partitioning tool written with ncurses libraries.
  MBR only.
- gdisk — GPT version of fdisk.
- cgdisk — GPT version of cfdisk.

## Partition alignment

Way back in the MSDOS days, a BIOS interacted with drives by knowing the
exact geometry of the drive, namely how many cylinders, heads, and
sectors were on a disk (**CHS** Addressing). Disks had physical sectors
of 512 bytes. This became a standard.

When newer, larger drives appeared, the number of sectors per track
varied (tracks on the outside of the platter can hold more sectors than
those on the inside), this broke CHS.

So **LBA** (Logical Block Address) was born. LBA holds the number of
sectors per track constant at 63, and varies the number of heads and
cylinders, which works for the old-school BIOSes.

The first partition always began at sector 64 (LBA 63. LBA starts at
zero). This maintained compatibility with MSDOS, BIOS and the
partitioning tools of the day.

Unfortuantely the decision to set sectors per track to 63 later became a
problem. The number 63 is not a power of 2.

Newer hard drives have a physical sector size of 4,096 bytes, and
emulate a sector size of 512 bytes (logical sector size) to maintain
backward compatibility. File systems such as ext4 and Windows NTFS use
block sizes of 4096. So even a one-byte file will consume 4096 bytes,
one physical sector, 8 logical sectors. So the idea is to get the
physical sectors to align with the file system's block size for best
performance.

If the first partition starts at LBA 63, it straddles 2 4096B physical
sectors. This means every block that gets written needs to write 2
physical sectors and this harms performance. The problem is even worse
with SSD drives some of which use 128kB "sectors".

To avoid these problems, alignment at one-megabyte boundaries is
recommended. That is, 2048 logical 512 byte sectors.

Newer version of Windows (Windows Vista, Windows 7 and Windows Server
2008) perform the following, reasonable alignment:

- Disk sizes less than or equal to four gigabytes should be aligned on
  sixty-four-kilobyte boundaries
- Disk sizes larger than four gigabytes should be aligned on
  one-megabyte boundaries

In Linux, older version of fdisk prior to verion 2.17.1 ( Feb. 2010)
defaulted to msdos mode. Misalignment will occur using fdisk, if special
parameters are not used. The newer fdisk from Version 2.17.1 will align
on the one megabyte boundary by default.

These days it is recommended to use gparted or parted for partitioning,
they handle alignment correctly and can do pretty much everything you
need.

### Solid state drives

Solid state drives are based on flash memory, and thus differ
significantly from hard drives. While reading remains possible in a
random access fashion, erasure (hence rewriting and random writing) is
possible only by whole blocks. Additionally, the erase block size (EBS)
are significantly greater than regular block size, for example 128KiB
vs. 4KiB, so it is necessary to align to multiples of EBS.

Partitioning tools:

In past, proper alignment required manual calculation and intervention
when partitioning. Many of the common partition tools now handle
partition alignment automatically:

    fdisk
    gdisk
    gparted
    parted

To verify a partition is aligned, query it using *blockdev* as shown
below, if a '0' is returned, the partition is aligned:

    sudo blockdev --getalignoff /dev/<partition>
    0

## Partitions larger than 2TB
- You cannot use fdisk to do this. You need to use the *parted* command.
- Before creating the partition, set the disk label to GPT (GUID
  partition table format).
- Use parted’s mklabel command to set disk label to GPT as shown below.


    # You probably need to be root to do this, so we add sudo
    sudo parted /dev/sdb

    GNU Parted 2.1
    Using /dev/sdb
    Welcome to GNU Parted! Type 'help' to view a list of commands.

    (parted) print
    Error: /dev/sdb: unrecognised disk label

    (parted) mklabel gpt

    (parted) print
    Model: Unknown (unknown)
    Disk /dev/sdb: 5909GB
    Sector size (logical/physical): 512B/512B
    Partition Table: gpt
    Number  Start  End  Size  File system  Name  Flags

For example: Create a 6TB Partition using parted mkpart

    # parted /dev/sdb

    (parted) mkpart primary 0GB 5909GB

    (parted) print
    Model: Unknown (unknown)
    Disk /dev/sdb: 5909GB
    Sector size (logical/physical): 512B/512B
    Partition Table: gpt

    Number  Start   End     Size    File system  Name     Flags
     1      1049kB  5909GB  5909GB               primary

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

    apropos mkfs

    mkfs.bfs (8)         - make an SCO bfs filesystem
    mkfs.ext2 (8)        - create an ext2/ext3/ext4 filesystem
    mkfs.ext3 (8)        - create an ext2/ext3/ext4 filesystem
    mkfs.ext4 (8)        - create an ext2/ext3/ext4 filesystem
    mkfs.ext4dev (8)     - create an ext2/ext3/ext4 filesystem
    mkfs.fat (8)         - create an MS-DOS filesystem under Linux
    mkfs.jfs (8)         - create a JFS formatted partition
    mkfs.minix (8)       - make a Minix filesystem
    mkfs.msdos (8)       - create an MS-DOS filesystem under Linux
    mkfs.ntfs (8)        - create an NTFS file system
    mkfs.reiserfs (8)    - The create tool for the Linux ReiserFS filesystem.
    mkfs.vfat (8)        - create an MS-DOS filesystem under Linux
    mkfs.xfs (8)         - construct an XFS filesystem

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

MBR partition table:

    Backup
    # sfdisk -d /dev/sda > /mount/usb/sda_pt

    Restore
    # sfdisk /dev/sda < /mount/usb/sda_pt

    Replicate /dev/sda to dev/sdd (for RAID array)
    # sfdisk -d /dev/sda | sfdisk -f /dev/sdd

GPT partition table:

    Backup
    # sgdisk --backup=/media/usb/sda_gpt /dev/sda

    Restore
    # sgdisk --load-backup=/media/usb/sda_gpt /dev/sda

    Replicate /dev/sda to dev/sdd (for RAID array)
    # sgdisk -R /dev/sdd /dev/sda
