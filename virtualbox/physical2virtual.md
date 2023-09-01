## Convert Windows Physical Machine to VM

- Clean up the Windows system. Get rid of junk files. Use something like
  Glary Utilities.
- Get the System UUID:


    wmic csproduct get UUID

- Get MAC address of network card:


    ipconfig /all - Physical address of active adapter.

- Save the UUID and MAC address to a file or piece of paper.


- Get Disk2vhd:
  <https://docs.microsoft.com/en-us/sysinternals/downloads/disk2vhd>
- Run Disk2Vhd, select "Use Vhdx" and "Use Volume Shadow Copy"
- Include all volumes
- Save the vhdx image file to the USB drive


- Move the vhdx file to the VirtualBox host machine.
- Convert the vhdx file to a vdi file:


    VBoxManage clonehd <filename>.vhdx <filename>.vdi --format VDI

- Create a new Virtual Machine.
- Do NOT add a Virtual Hard disk at this time.
- This will create a new directory.


- Move the vdi file into this new directory
- Now add the vdi file to the VM
- Configure networking, change the mac address to be same as physical
  machine. No dashes or colons in mac address.
- Finish configuring the VM, but do not start yet.
- Exit out of VirtualBox Manager GUI


- Edit the .vbox file
- Find the line `<Machine uuid="{blah..blah..blah...`
- Change the uuid to the same as Physical machine
- Edit ~/.config/VirtualBox/VirtualBox.xml
- Find the `<MachineEntry uuid=` line for the VM.
- Change the uuid to the same as Physical machine

If you get Blue Screen of Death when running VM:

<https://forums.virtualbox.org/viewtopic.php?t=49954>

- The BSOD will only show briefly, then the VM reboots. To leave the
  BSOD visible:


    VBoxManage setextradata "VM name" "VBoxInternal/PDM/HaltOnReset" 1

- Most often the BSOD shows: Stop 0x7B" which is usually caused by an
  inaccessible boot disk.
- Try changing the VM's "Storage Devices" controller type, it defaults
  to SATA, try IDE
- If that doesn't work, remove the SATA controller and add a PIIX4, then
  add the VDI file under the PIIX4

### UEFI

UEFI includes an extra disk partition, besides the windows system
partition. During the process of Disk2vhd, the UEFI partition becomes a
"raw" partition, and loses its special UEFI characteristics. We need to
convert it back to UEFI.

- Set the virtual machine to boot from the windows installation iso.
- Boot the VM
- When it is asking for "Language to install," etc, press Shift+F10.
  This will open a command prompt.
- Enter the command: diskpart

This will put you in the diskpart tool

    list disk

There should be only disk 0

    select disk 0
    list vol

Determine which volume is the RAW FS. We will use volume 3 as an
example. Now, assign a drive letter and format the drive.

    select vol 3
    assign letter L:
    format fs=fat32 label="BOOT"
    exit

Now you are back at the command prompt. Switch to drive L: and create
the efi directory tree

    L:
    md efi
    cd efi
    md microsoft
    cd microsoft
    md boot
    cd boot

Now add the boot records:

    bootrec /fixboot
    bcdboot C:\Windows /l en-us /s L: /f ALL

Once this has completed successfully.

    exit

Poweroff the Virtual Machine.

Now you can go into the Settings for the VM.

Under `System > Motherboard:` ,enable this setting: "Enable EFI (special OSes only)"

Restart the VM, ensuring that it boots from the virtual drive. Windows
should boot normally.
