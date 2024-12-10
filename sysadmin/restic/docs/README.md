# Restic Notes

## Install / Upgrade Restic package
The restic package in the official Linux Mint/Ubuntu repository is out of date.

Go to: https://packages.debian.org/sid/amd64/restic/download

- Download the restic_xxxx_amd64.deb file into your Downloads folder
- In the gui file manager, double click on the .deb file
- Optionally, run `sudo dpkg -i ~/Downloads/restic*.deb`
- Once it is installed **Launch a terminal and run "setcap"**

```bash
# Allows restic to backup files and dirs owned by root, without sudo
sudo setcap cap_dac_read_search=+ep /usr/bin/restic
    
# Verify
getcap /usr/bin/restic
    
# you should see:
# /usr/bin/restic cap_dac_read_search=ep
```

## Menu Entry for Restic-ui
- go to ~/.local/share/applications
- create new file: restic-ui.desktop

Paste this: (replace CHANGEME with your username)

```
[Desktop Entry]
Version=1.0
Name=Restic UI
Comment=Run Restic tui 
Exec=/home/CHANGEME/restic/Restic-ui
Icon=/usr/share/icons/Papirus/22x22@2x/apps/restic.svg
Terminal=true
Type=Application
Categories=Other
```

In the "Exec" line, you cannot use tricks like ~ or $HOME, you must use a Full Path.
