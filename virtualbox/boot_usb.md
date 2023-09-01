# Boot VM from USB drive

Add yourself to the "disk" group

    sudo usermod -a -G disk myusername

Log out and log in for this to take effect.

Create a "rawdisk" file that links to the usb device:

- In the terminal, change directory to your VirtualBox folder.
- Run the VBoxManage command to link the USB Drive to a vmdk file
  (Virtual Machine Disk):
- For example, if the USB device is /dev/sdb

<!-- -->

    VBoxManage internalcommands createrawvmdk -filename "usb.vmdk" -rawdisk /dev/sdb

- This creates a small file, `usb.vmdk`
- it's just a link to your USB drive.
- You just need to use it as primary hard drive (to boot on) for your
  virtual machine.

When rebooting the host, the usb drive is sometimes assigned to another
device.

- The usb vmdk file can be edited with a text editor
- just change the /dev/sdX to the correct letter, and "refresh" the VBox
  media manager.
