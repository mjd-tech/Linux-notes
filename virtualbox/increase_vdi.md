# Increase size of vdi file

- Resizing only works on **Dynamically allocated** vdi files
- Resizing **does not work** on **Fixed size** vdi files
- Resizing **does not work** if the guest has **snapshots**
- **Backup** the \*.vdi file before you start.

Increasing the drive does not increase the primary partition so this is
a **two step** process.

**Step 1:** Increase the size of the drive.

- You can do this in the GUI "Virtual Media Manager"
- Go in Hard Disks tab, select the vdi, click Properties
- There is a slider to adjust size

Or, from command line:

    VBoxManage modifyhd <absolute path to file> --resize <size in MB>

    Example: I have a 10GB drive that I want to be 20GB. the command would be:
    VBoxManage modifyhd "/path/to/file.vdi" --resize 20480

**Step 2:** Extend the primary partition to include the new drive space.

- Download the Gparted live cd or what ever partition manager program
  you wish and mount it to the guests virtual CD and boot the guest.
- From here you can expand the primary partition to use the new space.
- Windows Vista and newer guests can use the Disk Management to expand
  the primary partition.

Note: You **can not shrink** a guest drive with VirtualBox due to the
inherent danger of losing data or making the guest non-bootable.

What if I used Fixed Disks or Snapshots, or VMDK?

As the advice above states, resizing of fixed VDIs or VHDs is not
directly supported, nor is resizing of formats other than VHD/VDI, nor
can you easily resize disks which are part of a snapshot chain.

However, all of these problems are easily addressed if you clone the
disk to a supported format first, using :- (fields in brackets are
placeholders which should be replaced with actual filenames, the
brackets are not literal)

    VBoxManage clonehd <infilename or UUID> <outfilename> --format VDI --variant Standard

You can then resize the resulting dynamic VDI using "VBoxManage
modifyhd" as described in the previous message.

If a snapshot chain is involved then \<infilename\> should be the name
of the latest snapshot VDI in the "Snapshots" subfolder. Do not make the
rookie mistake of cloning the base VDI. In this case "clonehd" will
create a merged clone and it's important that you not incorporate this
back into a VM which is still expecting a chain of difference disk
images. Either build a new VM around the clone, or delete all the
snapshot markers from the original VM, then replace the disk file. If
the VM did not use difference images (no immutable drive, linked clone
or snapshot) then you can use the Storage settings panel to remove the
old disk and replace with the new one.
