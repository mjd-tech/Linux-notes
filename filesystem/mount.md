# Mount and umount
Different "filesystems" can be "mounted" into directories.

## Examples

Examples. replace /dev/sda with actual device or UUID

```
# mount will guess the filesystem type by default
sudo mount /dev/sda1 /mnt

# unmount it. notice it's "umount" not "unmount"
sudo umount /mnt

# specify the filesystem type explicitly
sudo mount /dev/sda2 -t ntfs-3g /mnt/ntfs 

# mount a filesystem image. uses a "loopback" mount
sudo mount Filesystem.img /home/user/MyFilesystem -o loop

# mount part of filesystem to another location, acts like a symlink, but not exactly
sudo mount --bind /var/www/html  /home/chroot/user/website

# another way
sudo mount -o bind /var/www/html  /home/chroot/user/website
```

## Mount USB drive at boot
USB drives get auto-mounted in a Desktop system, but you have to be logged into the system.

- To mount a USB drive automatically at boot, you need to edit `/etc/fstab`
- If the USB drive is NTFS, make sure you have the **ntfs-3g** package installed.

```
# Create a mount point.
sudo mkdir /mnt/usb1

# Plug in the drive and get the UUID.
lsblk -f

# Edit /etc/fstab as root
UUID=paste-uuid-here  /mnt/usb1  auto  defaults,noatime,nofail,x-systemd.device-timeout=15 0 2

# OR mount by LABEL.  can use e2label /dev/whatever label-here)
LABEL=label-here  /mnt/usb1  auto  defaults,noatime,nofail,x-systemd.device-timeout=15 0 2
# mount it

sudo mount -a
```

- nofail allows system to boot if USB drive not present.
- x-systemd.device-timeout=15 ... If you don't set this, it will wait 90 sec if no USB present
- `mount -a` mounts everything in /etc/fstab that isn't already mounted.

Beware the following scenario:
- You format an ext4 partition on a USB drive, and mount it using fstab.
- However, you can't write to it. only root can.
- The partition was formatted by root, and it's owned by root
- To fix, first **make sure the the partition is mounted**. ie. `mountpoint /mnt/usb1`
- Next chown the partition. ie. `sudo chown fred:fred /mnt/usb1`
- Now chmod the mounted partition. ie. `chmod 777 /mnt/usb1`

Note:
- If you do chown/chmod when the drive is NOT mounted, this won't fix the problem.
- The owner/perms of the mount point disappear when mounting something on it,
- and are replaced by the owner/perms of the mounted partition.
- The mount point's properties reappear when unmounting.

## fstab

**Warning:** Back up fstab before editing.

| Field       | Description                                                                                                                                |
|-------------|----------------------------------------------------------------|
| device      | The device/partition (by /dev location or UUID)                |
| mount point | use "swap" for swap file/partition. Avoid spaces in pathnames. |
| file system | btrfs, ext4, vfat, ntfs, sshfs, etc                            |
| options     | (see the man page for mount).                                  |
| dump        | This field should be set to 0, it's outdated...                |
| fsck order  | root device should be 1. Otherwise 2, or 0 to disable checking.|

### device

List devices by UUID

    lsblk -f    # also lsblk --fs
    
    # or
    sudo blkid

Ways to specify device:

- Device : /dev/sdxy
- Label : LABEL=label
- Samba : //server/share
- NFS : server:/share
- SSHFS : sshfs#user@server:/share

### Mount point

On desktop systems, USB drives are mounted in `/run/media/username/drive_id`, 
where drive_id is a LABEL or UUID.
Previously, the mount point was under `/media`, but not lately.

`/mnt` is the "official" place to mount stuff.

Typically you create a directory under /mnt


### File System Type

You may either use `auto` or specify a file system. Auto will attempt to
automatically detect the file system of the target file system and in
general works well. In general auto is used for removable devices and a
specific file system or network protocol for network shares.

Examples:

- auto - auto detect file system
- vfat - used for FAT partitions.
- ntfs, ntfs-3g - used for ntfs partitions.
- btrfs, ext4, etc.
- udf,iso9660 - for CD/DVD
- swap

### Options

Options are dependent on the file system.

Typical options:

    defaults - Equivalent to rw, suid, dev, exec, auto, nouser, async.
    noatime - don't update access time. recommended
    relatime - only update acess time once a day. This is default, but noatime recommended.
    sync/async - All I/O to the file system should be done (a)synchronously.
    exec/noexec - Permit/Prevent the execution of binaries from the filesystem.
    suid/nosuid - Permit/Block the operation of suid, and sgid bits.
    ro - Mount read-only.
    rw - Mount read-write.
    user - Permit any user to mount the filesystem. This automatically implies noexec, nosuid,nodev unless overridden.
    nouser - Only permit root to mount the filesystem. This is also a default setting.
    netdev - this is a network device, mount it after bringing up the network. Only valid with fstype nfs. 
    nofail - allows system to boot if drive not present. Useful for USB or network drives.

For specific options with specific file systems see:

      man mount

### Dump

Set this to 0. This is a relic from the past when people used `dump` to backup the system.
Dump is seldom used these days.

### Pass (fsck order)

Tells fsck what order to check the file systems.

    0 - do not check. Use for network shares, sshfs, smb, nfs, etc
    1 - check this partition first. Use for root partition.
    2 - check this partition next - For any other drives. 

### fstab Example

```
# /etc/fstab: static file system information.
#
# <file system>                           <mount point>  <type>  <options>  <dump>  <pass>
UUID=7F95-E677                            /boot/efi      vfat    umask=0077 0 2
UUID=68a383a9-4633-43c9-8ded-2a1fa6023b63 /              btrfs   subvol=/@,defaults,noatime,discard=async,ssd 0 0
UUID=68a383a9-4633-43c9-8ded-2a1fa6023b63 /home          btrfs   subvol=/@home,defaults,noatime,discard=async,ssd 0 0
UUID=68a383a9-4633-43c9-8ded-2a1fa6023b63 /var/cache     btrfs   subvol=/@cache,defaults,noatime,discard=async,ssd 0 0
UUID=68a383a9-4633-43c9-8ded-2a1fa6023b63 /var/log       btrfs   subvol=/@log,defaults,noatime,discard=async,ssd 0 0
UUID=8b8a969e-2210-466b-a671-40e369480373 swap           swap    defaults,noatime 0 0
tmpfs                                     /tmp           tmpfs   defaults,noatime,mode=1777 0 0

# An ext4 root partition
UUID=be35a709-c787-4198-a903-d5fdc80ab2f8 /              ext4    noatime,errors=remount-ro  0  1

# CD/DVD
/dev/scd0  /media/cdrom0  udf,iso9660  user,noauto,exec,utf8  0  0

# FAT  (Linux calls FAT file systems vfat)
UUID=12102C02102CEB83  /media/windows  vfat auto,users,uid=1000,gid=100,dmask=027,fmask=137,utf8  0  0

# NTFS - Use ntfs-3g for write access (rw) 
UUID=12102C02102CEB83  /media/windows  ntfs-3g  auto,users,uid=1000,gid=1000,dmask=027,fmask=137,utf8  0  0

# Samba
//server/share  /media/samba  cifs  user=sambauser,uid=1000,gid=100  0  0
# This asks for a password when mounting. To avoid, use a credentials file.
# Replace "user=sambauser" with "credentials=/etc/samba/credentials"
# In the credentials file, put two lines:
# username=sambauser
# password=password
# Then: sudo chown root.root /etc/samba/credentials && sudo chmod 400 /etc/samba/credentials

# NFS
Server:/share  /mnt/mynfs  nfs  rsize=8192,wsize=8192,noexec,nosuid
# your user id needs to be the same on the server and the workstation

# SSHFS
sshfs#user@server:/share  /mnt/mysshfs fuse  user,allow_other  0  0
# use public key authentication. your private ssh key needs to be copied to /root/.ssh
# because root will be making the connection
```

## Useful Commands

View the contents of /etc/fstab

    cat /etc/fstab

List UUIDs

    sudo blkid
    # Or
    ls -l /dev/disk/by-uuid

List drives and partitions

    sudo fdisk -l

List mounted partitions

    sudo mount

Mount all file systems in /etc/fstab

    sudo mount -a


## gvfs

- Ubuntu uses gvfs-fuse to mount network devices via SMB, SSH, NFS, etc.
- These devices can be mounted by a non-root user.
- The mount point is `/run/user/user_id/gvfs/share_name`
- You can use the gui file manager to mount network drives.

For command line use, use the gfs-mount command:

    gvfs-mount smb://WORKGROUP\;osmc@pi/USBdrive
    
    # This creates a mount point of:
    /run/user/1000/gvfs/smb-share:server=pi,share=usbdrive

It also creates an "alias" that can be used in the gui file manager: `smb://pi/usbdrive/`

    # List existing mounts
    gvfs-mount -l
    ...
    Mount(0): usbdrive on pi -> smb://pi/usbdrive/
    Type: GDaemonMount

However, this does not show the mount point. For that, you need to look
in the /run/user/1000/gvfs directory.

Note: gvfs-mount does not work correctly in an ssh session. Here's what
you have to do:

    export $(dbus-launch)
    /usr/lib/gvfs/gvfsd-fuse ~/.gvfs
    gvfs-mount smb://pi/USBdrive

Notice this uses the ~/.gvfs directory for the mount point. You cannot
use /run/user/1000/gvfs because you will get "permission denied", even
if you use sudo. Also, it is very slow to do the mount, for some unknown
reason...
