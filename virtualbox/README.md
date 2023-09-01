# VirtualBox

## Installation on Ubuntu

    sudo apt update
    sudo apt install virtualbox virtualbox-ext-pack virtualbox-guest-additions-iso
    sudo adduser $USER vboxusers

## Ubuntu Kernels

There are 2 kernel "tracks"

1.  General Availability (GA) kernel, i.e. the most stable kernel, which
    does not get updated to point releases. ex. 5.4 to 5.5 is a point
    release. 5.4.0-84 to 5.4.0-88 is not.
2.  Hardware Enablement (HWE) kernel, i.e. the most recent kernel
    released.

By Default, Ubuntu desktop systems follow the HWE track, to provide the
latest kernel, to support the latest hardware.

This can cause problems with VirtualBox, it needs to compile a kernel
module based on the kernel point release you are running.

- Older VirtualBox versions do not support newer kernel point releases,
  they need to be in sync.
- For some reason, Ubuntu puts the new hwe kernel in the repository but
  forgets to put the newest VirtualBox in the repo.
- VirtualBox cannot compile the the necessary kernel module, thus it
  breaks.

The easiest fix is to stay with the GA kernel instead of the HWE kernel.

For Ubuntu 20.04

    sudo apt remove linux-{image,headers}-generic-hwe-20.04
    sudo apt install linux-generic

If you also remove the already installed kernel packages that are later
than 5.4.x.y , it will stick with the 5.4 kernels. They will get updates
(bug fixes and security) till the end of life of the 20.04 release.

## Arch Linux (Manjaro)

Like Ubuntu, it's best to stay with a LTS kernel.

To make USB devices available, user should also be in the *storage*
group in addition to *vboxusers*

    sudo usermod -aG vboxusers $USER
    sudo usermod -aG storage $USER
