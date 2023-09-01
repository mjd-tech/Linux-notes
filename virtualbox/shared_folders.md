# Shared Folders (Host to Guest)

- Requires VirtualBox Guest Additions installed in Guest.
- Create the shared folder on the Host using VirtualBox Manager GUI


- Windows Guest: shared folders appear as a "Network" connection.
- Linux Guest: shared folders are mounted to `/media/sf_sharename` Where
  sharename is the name given to the Shared Folder

Linux Guest:

- Add your user to the vboxsf group.


    sudo usermod -a -G vboxsf $(whoami)

- Change the permissions of the mount point:


    sudo chown -R $(whoami):users /media/sf_sharename/

- log out and log in, or reboot guest. you should have the shared folder
  working.
