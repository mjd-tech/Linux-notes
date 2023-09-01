# ISO images

An ISO image is an archive file of an optical disc, composed of the data
contents from every written sector on an optical disc, including the
optical disc file system. ISO image files usually have a file extension
of .iso. The name ISO is taken from the ISO 9660 file system used with
CD-ROM media, but it also might also contain a UDF (ISO/IEC 13346) file
system (commonly used by DVDs and Blu-ray Discs).

ISO images are commonly used when downloading Linux distributions. The
image can be used as is, or burned to CDROM or DVD or USB stick.

## Verifying a Downloaded image

After downloading the image file, it is highly recommended that you
verify the md5 sum or sha256 sum (hash) of the .iso file. For this
example, we download an Ubuntu image and check the md5 sum.

    cd ~/Downloads    # for example
    wget http://releases.ubuntu.com/trusty/ubuntu-14.04.3-desktop-amd64.iso

### Manual method

Run the following command from within the download directory. It takes a
while depending on size of iso and speed of machine.

    md5sum ubuntu-14.04.3-desktop-amd64.iso

md5sum should then print out a single line after calculating the hash:

    cab6dd5ee6d649ed1b24e807c877c0ae ubuntu-14.04.3-desktop-amd64.iso

Compare the hash (the alphanumeric string on left) that your machine
calculated with the corresponding hash on the
[UbuntuHashes](https://help.ubuntu.com/community/UbuntuHashes) page.

An easy way to do this is to open the UbuntuHashes page in your browser,
then copy the hash your machine calculated from the terminal into the
"Find" box in your browser (in Firefox you can open the "Find" box by
pressing `Ctrl F`

When both hashes match exactly then the downloaded file is almost
certainly intact. If the hashes do not match, then there was a problem
with either the download or a problem with the server.

### Semi-automatic method

Ubuntu distributes the MD5 hashes in a file called MD5SUMS near the
bottom of the download page for your release
<http://releases.ubuntu.com>.

Download the MD5SUMS file to the same directory as the iso. Then run the
following in a terminal.

    md5sum -c MD5SUMS | grep -i 'OK$'

You should see:

    ubuntu-14.04.3-desktop-amd64.iso: OK

If you see nothing, it failed.

## Mounting an ISO image

Most Linux Desktop environments can mount an ISO image via the file
manager. To mount an ISO image using command line:

    # you probably need to be root to do this, so we add sudo
    sudo mount -t iso9660 -o ro,loop /path/to/file.iso /mount-point

    # Unmount - there is no "n" in the umount command!
    umount /mount-point

## Make ISO image from existing files

The simplest way to create an ISO image is to first copy the needed
files to one directory, for example: ./for_iso.

Then generate the image file with genisoimage:

    genisoimage -V "NAME_OF_ARCHIVE" -J -r -o isoimage.iso ./for_iso

Basic options

| Option       | Description                                                                                                                          |
|--------------|--------------------------------------------------------------------------------------------------------------------------------------|
| -V           | Volume Label. ISO 9660 standard imposes a limit of 32-character string length, limited to sets of: "A" to "Z", "0" to "9", and "\_". |
| -J           | Enables Joliet extension. Allocates special space to store file names in Unicode (up to 64 UTF-16 characters for each file)          |
| -joliet-long | Increases max length of file names from 64 to 103 characters. Non-compliant to Joliet specs and not commonly supported               |
| -r           | Enables Rock Ridge extension. Adds POSIX file system semantics, supports 255-character filenames and Unix file permissions           |
| -o           | Sets the file path for the resulting ISO image                                                                                       |

It is also possible to let genisoimage to collect files and directories
from various paths, and map them to more convenient locations on the
resulting image.

    genisoimage -V "BACKUP_2013_07_27" -J -r -o backup_2013_07_27.iso \
    -graft-points \
    /photos=/home/user/photos \
    /mail=/home/user/mail \
    /photos/holidays=/home/user/holidays/photos

## Add files to iso image

Reference:  
<http://bencane.com/2013/06/12/mkisofs-repackaging-a-linux-install-iso/>

- Mount the ISO as a directory


    sudo mkdir -p /mnt/iso
    sudo mount /dev/cdrom /mnt/iso
    --or--
    sudo mount -o loop /path/to/your.iso /mnt/iso 

- Copy the contents to a working directory


    cp -r /mnt/iso /tmp/newiso

- Make your changes
- After modifications are done, create new ISO image


    cd /tmp/newiso
    sudo mkisofs -o /tmp/new.iso -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -J -R -V "new iso name" .

- After mkisofs is finished new ISO file will be created at /tmp
  directory
- It works in any Linux distributions like Ubuntu, Debian or Fedora.
