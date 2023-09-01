# Copy Remmina Settings to new install

<https://askubuntu.com/questions/420986/copy-or-export-remmina-remote-desktop-files-to-another-ubuntu-install>

Backup with:

    $ cd ~
    $ tar -czf remmina_backup.tgz .local/share/remmina/ .config/remmina/ .local/share/keyrings .ssh/

Restore with:

    $ cd ~
    $ tar -xf remmina_backup.tgz

Snap users Backup with:

    $ cd ~
    $ tar -czf remmina_backup.tgz snap/remmina/<version>/.local/share/remmina/ snap/remmina/<version>/.config/remmina/ .local/share/keyrings .ssh/
