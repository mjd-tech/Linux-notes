# File System
```
lsblk                 disks and partitions, tree view, with size, mounts
sudo blkid            partitions, with LABEL, UUID, TYPE (ex. btrfs)
mount                 usually too much information
mount | grep '^/dev'  usually what you want
/etc/fstab            lists what gets mounted at boot
sudo fdisk -l         list partition table
sudo fdisk /dev/sda   add/delete partitions
gparted               gui partition editor

# backup GPT partition table
sfdisk -d /dev/sdX > part_table
# restore
sfdisk /dev/sdX < part_table
```

```bash title=/etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a device; this may
# be used with UUID= as a more robust way to name devices that works even if
# disks are added and removed. See fstab(5).
#
# <file system>             <mount point>  <type>  <options>  <dump>  <pass>
UUID=7F95-E677                            /boot/efi      vfat    umask=0077 0 2
UUID=68a383a9-4633-43c9-8ded-2a1fa6023b63 /              btrfs   subvol=/@,defaults,noatime,discard=async,ssd 0 0
UUID=68a383a9-4633-43c9-8ded-2a1fa6023b63 /home          btrfs   subvol=/@home,defaults,noatime,discard=async,ssd 0 0
UUID=68a383a9-4633-43c9-8ded-2a1fa6023b63 /var/cache     btrfs   subvol=/@cache,defaults,noatime,discard=async,ssd 0 0
UUID=68a383a9-4633-43c9-8ded-2a1fa6023b63 /var/log       btrfs   subvol=/@log,defaults,noatime,discard=async,ssd 0 0
UUID=8b8a969e-2210-466b-a671-40e369480373 swap           swap    defaults,noatime 0 0
tmpfs                                     /tmp           tmpfs   defaults,noatime,mode=1777 0 0
```

## Overview

- The Linux kernel establishes the root / filesystem at bootup. It is unpopulated.
- A **device**, (drive or partition), contains **a** filesystem. ie. btrfs, ext4, ntfs, fat, etc.
- Devices are **mounted** into / in order to get a functional system.
- All the mounted devices become **the** filesystem.

## Comparison with MS Windows

- Windows traditionally assigns a "drive letter" (C: D: E: etc.) to each "drive".
- Each of these drive letters becomes a separate hierarchy.
- Linux has only **one** hierarchy, beginning at /
- It is possible in Windows to mount a drive into a directory, without assigning a new drive letter.
- However, this is not common practice.

## Virtual Filesystems

In Linux, "everything is a file". There are special filesystems which are dynamically populated by the
linux kernel during runtime.

- /dev - devices such as hard drives, CD, DVD, USB drives
- /proc - process information
- /sys - dynamic system info
- /run - runtime info for processes.

Note:
- /dev and /proc originated in Unix.
- Linux greatly extended /proc and created a mess.
- /sys and /run were added to better organize things.
- "dynamically populated" directories should be excluded in backup scripts.

## How files are stored

1.  Directories - contain the *name* of the file and its associated *inode*
2.  Inodes - (Index Node) contain "metadata" about the file
3.  Data - the actual data in the file

### Directories
- Directories are special files that map filenames to inodes.

### inodes
- file size
- User / Group ID of the file's owner
- Permissions
- Additional system and user flags
- Timestamps
- A link counter that lists how many hard links point to the inode
- Pointers to the disk blocks that store the file’s contents

inodes do **not** contain the filename or data.

