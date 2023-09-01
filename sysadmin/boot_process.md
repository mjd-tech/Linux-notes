# Boot Process
Reference: 
- [rodsbooks](https://www.rodsbooks.com/efi-bootloaders/principles.html)
- [Arch Linux](https://wiki.archlinux.org/title/Arch_boot_process#Under_UEFI)

## UEFI Summary
- Needs "EFI" partiton, FAT32 200Mb or more.
- EFI partition stores "bootloaders" in the EFI directory.
- bootloaders are installed in subdirectories, typically named after the OS that created them
- For example: Ubuntu, Fedora, Manjaro, Microsoft
- Also has has a "fallback" directory "boot" with a fallback bootloader "bootx64.efi"

bootx64.efi was originally intended for use only on removable media,
so that they could be booted to install an OS.  
It was quickly adopted as a fallback for hard disk installations, as well.  
This way, the computer will remain bootable if the NVRAM-based boot manager list is damaged.  
In practice, some EFIs have badly broken boot managers that tend to forget
or ignore their boot loaders. These computers boot reliably only from the fallback boot loader.

Most linux systems use Grub bootloader.
- UEFI executes EFI/ubuntu/grubx64.efi
- grub knows how to read ext4, btrfs, etc. so it looks in /boot/grub
- in /boot/grub there's a bunch of kernel modules and config files
- grub puts a menu on the screen (or not)
- Make your selection
- grub loads "initial ramdrive". Example: initrd.img-5.15.10... initramfs-5.15.10...
- then the actual kernel. Example: vmlinuz-5.15.10...

Ramdrive
- initrd.img or initramfs is cpio archive file.
- It holds all the drivers and modules your kernel requires in ram.
- So the kernel does not need to grab those things off of your relatively slow SSD or hard drive. 
