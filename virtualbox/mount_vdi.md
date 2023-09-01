# Mount a VDI file in Ubuntu

Note: this is a **read-only** mount

A VDI (Virtual Disk Image) file can contain more than one "Drive" or
"Volume"

In this example we want to mount the "C Drive" of a Windows 10 VDI.

First we need to get the UUID of the VDI file.

    vboximg-mount --list

    # example output
    -----------------------------------------------------------------
    VM:   Windows 10 Pro
    UUID: 66f311a3-e0ee-4666-a905-880a94928cbe

        Image:   Windows 10 Pro.vdi
        UUID:    95f4db95-0eab-4d28-8bbc-72592fb957cd

In this example, you want the UUID of the **image** (begins with 95),
**not** the UUID of the VM (begins with 66)

Create a mount point in your home directory, for example vbox_vdi:

    mkdir ~/vbox_vdi

Mount the vdi (remember to change the UUID to yours):

    vboximg-mount -i 95f4db95-0eab-4d28-8bbc-72592fb957cd -o allow_root ~/vbox_vdi

NOTE: You may need to edit the “/etc/fuse.conf” to make the -o
allow_root flag work.

    sudo nano /etc/fuse.conf
    # Uncomment the "user_allow_other" line.

List the volumes contained in the vdi file:

    ls -Alh ~/vbox_vdi/
    total 29G
    -rw-r--r-- 1 nobody nogroup  50G Oct  4 09:54  vhdd
    -rw-rw-rw- 1 root   root    549M Dec 31  1969  vol0
    -rw-rw-rw- 1 root   root     49G Dec 31  1969  vol1
    -rw-rw-rw- 1 root   root    516M Dec 31  1969  vol2
    lr--r--r-- 1 root   root       0 Sep 28 15:13 'Windows 10 Pro.vdi' -> '/home/username/VirtualBox VMs/Windows/Windows 10 Pro/Windows 10 Pro.vdi'

In this case, **vol1** is the one we want, it is the largest size, so
it's the "Drive C" NTFS Windows volume.

Create another mount point

    sudo mkdir /mnt/vbox_vol

Mount the Windows 10 C drive:

    sudo mount vbox_vdi/vol1 /mnt/vbox_vol

Finally go to /mnt directory and there you are. Unmount:

To un-mount the guest os file system, run command:

    sudo umount /mnt/vbox_vol

To un-mount the VBox disk image, run command:

    umount ~/vbox_vdi
