# Compact a Guest's VDI file

## Summary

      Zero fill the virtual machine’s disk using the dd command
      Shut down the machine
      Use Oracle’s VBoxManage program to compact the VDI
      Restart the virtual machine

## Linux Guest

Use the following command to zero-fill the disk:

    dd if=/dev/zero of=fillfile bs=1M

- dd will complain and quit when the virtual disk runs out of space
- Once dd has completed, delete your zero-fill file

<!-- -->

    rm fillfile

- shut down the virtual machine

## Windows Guest

- run the defragment utility.
- download SDelete from Sysinternals and run this:

<!-- -->

    sdelete –z
    or
    sdelete -z c:  (if it complains about needing a drive letter)

- shut down the virtual machine.

## Host

Run this command on the host:

    VBoxManage modifyhd /path/to/your.vdi --compact

- When the process is complete, you should have a smaller vdi.
- If not, something went wrong with the zero-filling process or there
  was no room to compact your VDI.
