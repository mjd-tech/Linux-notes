# mkfs and mkswap
make filesystem, make swap

## mkfs
Format a partition with a file system.

```
# Examples. replace /dev/sda with actual device
sudo mkfs -t ext4 /dev/sda2
# is the same as
sudo mkfs.ext4 /dev/sda2

# EFI partition
sudo mkfs.fat -F 32 /dev/sda1

```
To see what filesystems you can create on your system:

```
apropos mkfs
```
Note: mkfs.vfat and mkfs.msdos are both symlinks to mkfs.fat, they are the same utility.

## mkswap
Create a swap area on a partition or in a file.

### Swap partition

Typically set up during installation, or with fdisk or gparted.
The partition is set to type 82.

```
sudo mkswap /dev/sda2
# specify custom UUID:
mkswap -U custom_UUID /dev/sda2
# enable swap
swapon /dev/sda2
# disable swap:
swapoff /dev/sda2
```

Enable swap partition on boot:

```bash title=/etc/fstab
/dev/sda2 none swap sw 0 0
--or--
UUID=78fbb188-3643-445c-8aff-1a264d36190e none swap sw 0 0
```

## Swap file

- By default, Linux does not support dynamic swapfiles
- It can be done with 3rd party packages, but not advisable
- So you're stuck with a static swapfile.

Create swapfile:
```
# first create an empty file. example: 8GB
sudo fallocate -l 8G /swapfile
--or--
sudo dd if=/dev/zero of=/swapfile bs=1M count=8096

# set permissions
chmod 600 /swapfile

# enable it
mkswap /swapfile

# Activate it
swapon /swapfile
```
Enable swap file on boot:

```bash title=/etc/fstab
/swapfile none swap defaults 0 0
```

Remove a swap file:

```
swapoff -a
rm -f /swapfile

```

Finally remove the relevant entry from /etc/fstab.
