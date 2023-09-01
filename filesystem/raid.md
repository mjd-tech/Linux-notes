# Raid

:!: Make sure you know which drive you are dealing with, use badblocks

    badblocks /dev/sda

This will make the /dev/sda activity LED light up.

## Prepare Drives

Raid arrays will build much faster if drives are "zeroed out"

    dd if=/dev/zero of=/dev/sda bs=4M

The bs=4M (block size 4 Megabytes) may be too large. If there's a
problem try a smaller block size, like 1M. You want this as large as
possible to reduce the time it takes to zero the drive.

Create a single primary partition on each of /dev/sda /dev/sdb /dev/sdc
etc. from sector 2048 to the end of the disk, and set the partition type
to fd for Linux Raid Autodetection.

    fdisk /dev/sda

The keystroke-sequence in fdisk (if the disk is empty in the beginning,
meaning no partitions) is:

    n <return> p <return> 1 <return> 2048 <return> <return> t <return> fd <return> w <return>.

Rather than do this for each drive, you can copy the partition table
like this:

    sfdisk -d /dev/sda | sfdisk -L /dev/sdb

Dumps partition table of sda, copies to sdb, without complaining about
partitions being aligned with cylinders (irrelevant to Linux).

### Large Partitions

fdisk won't create partitions larger than 2 TB. To solve this problem
use GNU parted command with GPT. The GPT Partition Table is a part of
the Extensible Firmware Interface (EFI) standard, a replacement for the
outdated PC BIOS. EFI uses GPT where BIOS uses a Master Boot Record
(MBR).

Linux Create 3TB partition:

    parted -a optimal /dev/sdb
    (parted) mklabel gpt
    (parted) unit s
    (parted) mkpart primary 2048s 100%
    (parted) align-check opt 1
    1 aligned
    (parted) set 1 raid on
    (parted) quit

This starts 1mb off the front of the disk and confirms that the disk is
aligned.

## Create a new RAID array

    mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sda1 /dev/sdb1

or using the compact notation:

    mdadm -Cv /dev/md0 -l1 -n2 /dev/sd[ab]1

Using a "bitmap" can greatly increase the speed of rebuilding the Raid
array.

    mdadm --create /dev/md0 --bitmap=internal --level=1 -n 2 /dev/sd[ab]1

When creating a RAID 5 array with 4 or more disks, mdadm defaults to
creating the array with one drive as "spare". If you don't want this

    mdadm -Cv --force --assume-clean /dev/md0 --bitmap-internal --level=5 -n4 /dev/sd[bcde]1

## Write RAID configuration to config file

After we create our RAID arrays we add them to this file using:

    mdadm --detail --scan >> /etc/mdadm/mdadm.conf

## Remove a disk from an array

We can’t remove a disk directly from the array, unless it is failed, so
we first have to fail it (if the drive is already in failed state this
step is not needed):

    mdadm --fail /dev/md0 /dev/sda1

and now we can remove it:

    mdadm --remove /dev/md0 /dev/sda1

This can be done in a single step using:

    mdadm /dev/md0 --fail /dev/sda1 --remove /dev/sda1

## Add a disk to an existing array

After replacing the failed drive, add it back into the array:

    mdadm --add /dev/md0 /dev/sdb1

## Verifying the status of the RAID arrays

    cat /proc/mdstat

or

    mdadm --detail /dev/md0

The output of this command will look like:

    cat /proc/mdstat
    Personalities : [raid1]
    md0 : active raid1 sdb1[1] sda1[0]
    104320 blocks [2/2] [UU]

    md1 : active raid1 sdb3[1] sda3[0]
    19542976 blocks [2/2] [UF]

    md2 : active raid1 sdb4[1] sda4[0]
    223504192 blocks [2/2] [U_]

here we can see: md0 is working fine. md1 has a failed drive. md2 is
degraded with a missing drive.

## Monitor the status of a RAID rebuild operation

    watch cat /proc/mdstat

## Stop and delete a RAID array

If we want to completely remove a raid array we have to stop if first
and then remove it:

    mdadm --stop /dev/md0
    mdadm --remove /dev/md0

and finally we can even delete the superblock from the individual
drives:

    mdadm --zero-superblock /dev/sda

There are many other usages of mdadm particular for each type of RAID
level, and I would recommend to use the manual page (man mdadm) or the
help (mdadm –help) if you need more details on its usage. Hopefully
these quick examples will put you on the fast track with how mdadm
works.

## Misc

<http://askubuntu.com/questions/209702/why-is-my-raid-dev-md1-showing-up-as-dev-md126-is-mdadm-conf-being-ignored>

Here is an excerpt from the above link:

> In short, I chopped my /etc/mdadm/mdadm.conf definitions from:
> ARRAY /dev/md1 metadata=1.2 name=ion:1
> UUID=aa1f85b0:a2391657:cfd38029:772c560e
>
> ARRAY /dev/md2 metadata=1.2 name=ion:2
> UUID=528e5385:e61eaa4c:1db2dba7:44b556fb
> to:
> ARRAY /dev/md1 UUID=aa1f85b0:a2391657:cfd38029:772c560e
>
> ARRAY /dev/md2 UUID=528e5385:e61eaa4c:1db2dba7:44b556fb
> and ran:
> sudo update-initramfs -u
> I am far from an expert on this, but my understanding is this ...
> The kernel assembled the arrays prior to the normal time to assemble
> the arrays occurs. When the kernel assembles the arrays, it does not
> use mdadm.conf. Since the partitions had already been assembled by the
> kernel, the normal array assembly which uses mdadm.conf was skipped.
> Calling sudo update-initramfs -u tells the kernel take a look at the
> system again to figure out how to start up.
