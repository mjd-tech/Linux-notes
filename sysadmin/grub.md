# Grub Boot Loader
Grub2 is the default bootloader on most linux systems.
Some systems use systemd-boot.

## GRUB menu doesn't display
- If no other operating systems are installed, GRUB 2 will boot directly to Linux.
- Hold down (right) SHIFT to display the menu during boot. Or, try ESC.

## Make menu display and set the timeout
Edit (as root) `/etc/default/grub`
```
GRUB_TIMEOUT_STYLE=menu
GRUB_TIMEOUT=5
```
Then,
```bash
sudo update-grub
```
Note: `update-grub` is a "stub" for `grub-mkconfig -o /boot/grub/grub.cfg`

## Repair Grub
Example: you install Windows, and it replaces GRUB with its own boot loader.  
You need to replace the Windows boot loader with GRUB.

### Ubuntu
- Boot from an Ubuntu live USB, same version as what's installed.
- Make sure that you booted using EFI: `efibootmgr -v`

You need the following information:
1. The device name of the EFI partition. ex. `/dev/sda1`
2. The device name of the Linux system partition. ex. `/dev/sda2`
3. The filesystem of the Linux system partition. (`ext4` or `btrfs`)

Identify the EFI partition, and the Linux system partition
```bash
sudo fdisk -l | grep -Ei 'efi|Linux'
```
Identify the filesystem of the Linux system partition
```bash
sudo blkid
# or
df -Th
```

If you have an **ext4** system partition:

```bash
# Replace sda2 with your partition name
sudo mount /dev/sda2 /mnt
```
If you have a **btrfs** system partition:

```bash
# Replace sda2 with your partition name
sudo mount /dev/sda2 /mnt/ -t btrfs -o subvol=@
```

Next, mount the EFI partition:

```bash
# Replace sda1 with your partition name
sudo mount /dev/sda1 /mnt/boot/efi
```
Bind mount some other necessary stuff:
```bash
for i in /dev /dev/pts /proc /sys /run; do sudo mount -B $i /mnt$i; done
```
chroot into your installed Ubuntu system:
```bash
sudo chroot /mnt
```

At this point, you're in the installed system, not the live session, and running
as root, so you don't need sudo for the following commands.

Reinstall grub:

**Note:**
- Specify a **device** name like **/dev/sda**
- Do **NOT** specify a partition name like **/dev/sda1**  

```bash
grub-install /dev/sda  # replace sda with actual device name
# DO NOT specify a partition name.
# For example: /dev/sda1 is WRONG!!!
```

Now update the grub config
```bash
update-grub
```
If everything worked without errors, then you're all set:

```bash
# exit chroot
exit  # or Ctrl-d
sudo reboot
```

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

```bash
# Reinstall grub
pacman -Syu grub 
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=manjaro --recheck

# Update the grub configuration
grub-mkconfig -o /boot/grub/grub.cfg

# Exit chroot
exit
```

Troubleshooting:
```bash
# Verify the existance of an EFI system partition
lsblk -o PATH,PTTYPE,PARTTYPE,FSTYPE,PARTTYPENAME

# Verify the efi filesystem is loaded
ls /sys/firmware/efi
```
## log getting spammed with PCIe errors
For example:
```
PCIe Bus Error: severity=Corrected, type=Physical Layer, id=00e5(Receiver ID)
```
Fix it:
```
sudo vim /etc/default/grub
```
add pcie_aspm=off to GRUB_CMDLINE_LINUX_DEFAULT - something like this:

`GRUB_CMDLINE_LINUX_DEFAULT="quiet splash pcie_aspm=off"`


```
sudo update-grub
# reboot the system
```
