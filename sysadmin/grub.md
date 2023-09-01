# Grub Boot Loader
Grub2 is the default bootloader on most linux systems.
Some systems use systemd-boot.

## GRUB menu doesn't display
- If no other operating systems are installed, GRUB 2 will boot directly to Linux.
- Hold down (right) SHIFT to display the menu during boot. Or, try ESC.

## Set the menu timeout
Edit /etc/default/grub

    GRUB_TIMEOUT=5

Then,

    sudo update-grub

Note: `update-grub` is a "stub" for `grub-mkconfig -o /boot/grub/grub.cfg`

## Repair Grub
Example: you install Windows, and it replaces GRUB with its own boot loader.  
You need to replace the Windows boot loader with GRUB.

### Ubuntu
- Boot from an Ubuntu live USB, same version as what's installed.
- Determine your EFI partition and your linux system partition.
- Make sure that you booted using EFI: `efibootmgr -v`

try these commands
```
sudo fdisk -l
sudo blkid
df -Th
```

#### Ext4

```
# Mount system partition:
sudo mount /dev/sda2 /mnt  # Replace sda2 with your partition name

# Mount EFI partition, replacing sda1 with the actual partition name:
sudo mount /dev/sda1 /mnt/boot/efi
```

#### Btrfs
```
# Mount root subvolume. Replace sda2 with your partition name
sudo mount /dev/sda2 /mnt/ -t btrfs -o subvol=@

# Mount EFI subvolume. Replace sda1 with actual partition name
sudo mount /dev/sda1 /mnt/boot/efi
```

Both:
```
# Bind mount some other necessary stuff:
for i in /dev /dev/pts /proc /sys /run; do sudo mount -B $i /mnt/$i; done

# chroot into your installed Ubuntu system:
sudo chroot /mnt
```

At this point, you're in your install, not the live session, and running
as root. Reinstall and Update grub:

    grub-install /dev/sda  # replace sda with actual device name
    update-grub

Note: Do **NOT** use the partition number with the grub-install command.

If everything worked without errors, then you're all set:

      # exit chroot
      exit  # or Ctrl-d
      sudo reboot

At this point, you should be able to boot normally.

### Manjaro
https://wiki.manjaro.org/index.php?title=GRUB/Restore_the_GRUB_Bootloader
- Boot into a recent Live USB.
- Default usernames and passwords: **manjaro**
- Become root. Use manjaro-chroot
```
su
manjaro-chroot -a
```
This will scan your drive, find your Manjaro installation, 
read /etc/fstab, mount needed partitons, and put you in chroot.

```
# Reinstall grub
pacman -Syu grub 
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=manjaro --recheck

# Update the grub configuration
grub-mkconfig -o /boot/grub/grub.cfg

# Exit chroot
exit
```

Troubleshooting:
```
# Verify the existance of an EFI system partition
lsblk -o PATH,PTTYPE,PARTTYPE,FSTYPE,PARTTYPENAME

# Verify the efi filesystem is loaded
ls /sys/firmware/efi
```

