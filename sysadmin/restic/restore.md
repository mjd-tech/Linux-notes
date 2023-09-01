# Restore
Two ways
1. restic restore - to restore many files
2. restic mount - easiest way to restore just a few files

## restic restore
- get list of snapshots with id numbers `restic snapshots`
- restore a directory `restic restore 79766175:/home/fred/.thunderbird --target /home/fred/tmp`

## restic mount
- Mount the restic repository `restic mount --allow-other ~/restic/recovery`
- use gui file manager to drag and drop.
- for files/directories owned by root, right click...Open as Administrator
- or use midnight commander as root `sudo mc`

Standalone script, can run on same or another box (provided restic is installed).  
Modify as needed.
```
#!/bin/bash
# mount restic repo

# SET THESE TWO VARIABLES!
export RESTIC_REPOSITORY=sftp:fred@192.168.1.1:/full/path/to/restic/repo/
export RESTIC_PASSWORD=password

# Recovery directory
if ! [[ -d ~/restic/recovery ]]; then
    mkdir -p ~/restic/recovery || exit 1
fi

echo "Mounting restic repository (read-only) to ~/restic/recovery ..."
echo
restic mount --allow-other ~/restic/recovery
```

