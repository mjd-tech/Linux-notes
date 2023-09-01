# dd command
copy block-by-block, regardless of filesystem.

NOTE: dd is sometimes called "Disk Destroyer".  
**Make 100% absolutely sure you are using the correct DESTINATION device name!**  
There is NO "undo"

- For cloning partitions or entire drives, consider **Clonezilla**, which has a "wizard" that helps prevent errors.
- Otherwise, you need a bootable "Live" Linux usb, cd, dvd.
- Both source and destination partitions/drives should be **unmounted**
- Most dd operations need to be done as ROOT user
- The destination needs to be **the same size or bigger** than the source

TIP:  
USB drives usually have an LED that indicates activity. You can force activity on **unmounted** drive sdb (for example) with:

    badblocks /dev/sdb

badblocks searches a drive for bad blocks, but here it is used to merely indicate disk activity. Cancel it with Ctrl-C

## Cloning a partition
From physical disk /dev/sda, partition 1, to physical disk /dev/sdb, partition 1:

    dd if=/dev/sda1 of=/dev/sdb1 bs=64K conv=noerror,sync status=progress

Note: if the output partition of= (sdb1 in the example) does not exist, dd will create a file with this name and will start filling up your root file system.

## Cloning an entire disk
From physical disk /dev/sda to physical disk /dev/sdb:

    dd if=/dev/sda of=/dev/sdb bs=64K conv=noerror,sync status=progress

This will clone the entire drive, including the partition table, bootloader, all partitions, UUIDs, and data.

### Explanation
- `if=` input file, the "source"
- `of=` output file, the "destination" If you get if= and of= wrong, you will have **BIG PROBLEMS**
- `bs=` block size. 512 bytes by default, which is too small, it will take forever. Too big has problems too. Arch Wiki recommends 64k.
- `noerror` instructs dd to continue operation, ignoring all read errors. Default behavior for dd is to halt at any error.
- `sync` fills input blocks with zeroes at the end of the block if there were any read errors somewhere in the block, so data offsets stay in sync
- `status=progress` shows periodic transfer statistics which can be used to estimate when the operation may be complete.

## Backup a disk to a file
Since there may be a lot of unused space on the disk, compress the image file.

```
# Backup entire drive /dev/sda to a compressed image file
dd if=/dev/sda conv=sync,noerror bs=64K | gzip -c  > /path/to/backup.img.gz

# restore compressed image file to /dev/sda
gunzip -c /path/to/backup.img.gz | dd of=/dev/sda
```
Can also backup partitions like /dev/sda1 using this method.

## Wipe a drive
```
dd if=/dev/zero bs=5m of=/dev/sda
```

## Basic Syntax

    dd if= of= bs= count=

- "if" means "input file"
- "of" means "output file"
- "bs" means "block size" which defaults to 512 bytes. The block size is
  usually some power of 2, not less than 512 bytes. For example: 512,
  1024, 2048, 4096, 8192, 16384. It can however, be any reasonable
  number.
- "count" is how many times to do it. Often used with "block size" to
  copy a specific number of bytes.

## Cloning a disk
Clonezilla has an easy "wizard" to do this, but if for some reason you need to do it
manually, read on.

Boot into a "Live" Linux. ie Ubuntu, Linux Mint, etc

Make sure which is the source drive and which is the destination. This
usually involves mounting the drives and looking at the contents.
Usually the Live CD will mount the drives for you.

Before doing anything with dd, you need to unmount the drives. If no
obvious graphical way then

    umount /dev/sda

(Unmounts drive sda). Notice the command is **umount** not unmount.  
Be sure to unmount both the source and destination drives. You probably
need to be root to do this. It depends.

USB drives usually have an LED that indicates activity. You can force
activity on unmounted drive sdb (for example) with:

    badblocks /dev/sdb

badblocks searches a drive for bad blocks, but it is used here to merely
indicate disk activity. Cancel it with Ctrl-C

Once the drives are identified and unmounted, you can clone the disk.

Lets say the source is /dev/sda and the destination is /dev/sdb. Both
are unmounted. You will probably need to be root to do this.

    dd if=/dev/sda of=/dev/sdb bs=4096 conv=notrunc,noerror,sync

This will clone the entire drive, including MBR (and therefore
bootloader), all partitions, UUIDs, and data.

- notrunc or 'do not truncate' maintains data integrity by instructing
  dd not to truncate any data.
- noerror instructs dd to continue operation, ignoring all read errors.
  Default behavior for dd is to halt at any error.
- sync writes zeroes for read errors, so data offsets stay in sync.
- bs=4096 sets the block size to 4k, an optimal size for hard disk
  read/write efficiency and therefore, cloning speed.

If you get the if= and of= devices wrong, you will be seriously hosed.  
if= input file and of= output file. dd treats entire hard drives as a
"file"

## Showing Progress

dd gives no progress report, it just sits there and gives no feedback
whatsoever until it is finished. Open up a second terminal window and
run:

    pgrep '^dd$'

to find the process id (PID) of the dd process. Then, still in second
window do

    watch -n 10 kill -USR1 PID

where PID is the PID that you got from the pgrep command.  
Now go back to the first window where dd is running and you will get a
status update every 10 seconds.

## Backing up the MBR

The MBR is stored in the the first 512 bytes of the disk. It consist of
3 parts:

- The first 446 bytes contain the boot loader.
- The next 64 bytes contain the partition table (4 entries of 16 bytes
  each, one entry for each primary partition).
- The last 2 bytes contain an identifier

To save the MBR of drive /dev/sda into the file "mbr.img":

    dd if=/dev/sda of=mbr.img bs=512 count=1

To restore (be careful : this could destroy your existing partition
table and with it access to all data on the disk):

    dd if=mbr.img of=/dev/sda

If you only want to restore the boot loader, but not the primary
partition table entries, just restore the first 446 bytes of the MBR:

    dd if=mbr.img of=/dev/sda bs=446 count=1

To restore only the partition table, one must use

    dd if=mbr.img of=/dev/sda bs=1 skip=446 count=64

You can also get the MBR from a full dd disk image.

    dd if=/path/to/disk.img of=mbr.img bs=512 count=1

## Create Compressed Disk Image

- Boot from a liveCD or liveUSB.
- Make sure no partitions are mounted from the source drive
- Mount the destination drive
- Backup the source drive

<!-- -->

    dd if=/dev/sda conv=sync,noerror bs=64K | gzip -c  > /some/path/sda.img.gz

- Save extra information about the drive geometry necessary in order to
  interpret the partition table stored within the image. The most
  important of which is the cylinder size.

<!-- -->

    fdisk -l /dev/sda > /some/path/sda_fdisk.info

NOTE: You may wish to use a block size (bs=) that is equal to the amount
of cache on the HD you are backing up. For example, bs=8192K works for
an 8MB cache. The 64K mentioned in this article is better than the
default bs=512 bytes, but it will run faster with a larger bs=. Restore
system

## Restore from Compressed Disk Image

    gunzip -c /some/path/sda.img.gz | dd of=/dev/sda
