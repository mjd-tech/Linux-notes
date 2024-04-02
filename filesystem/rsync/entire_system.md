# Backup entire system

This example backs up the entire system to a USB drive mounted at
/media/usbdrive. Must be run as root.

    rsync -axAXv --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} / /media/usbdrive/

The command uses Bash "brace expansion" to auto-generate multiple
`--exclude=` expressions.  

If you are using /bin/sh, you cannot use brace expansion so you must use
explicit multiple --exclude statements. i.e.

    rsync -axAXv --exclude="/dev/*" --exclude="/proc/*" --exclude="/sys/*" ...

or use a file to store the excludes and use --exclude-from some_file.

- Using the -aAX set of options, the files are transferred in archive
  mode, ensuring that symbolic links, devices, permissions and
  ownerships, modification times, ACLs and extended attributes are
  preserved.
- The -x option is "don't cross file system boundaries". This means
  don't backup things "mounted" into your / filesystem like NFS, Samba,
  SSHFS mount points, DVD, CD-ROM, etc. Usually these things get mounted
  to /mnt or /media so they are already excluded by the above command.
  But it is a good idea to use the -x option, just in case some
  "external" file system is mounted somewhere other than /mnt or /media.
- The `--exclude` option will cause files that match the given patterns
  to be excluded.
- The _contents_ of /dev, /proc, /sys, /tmp and /run were excluded
  because they are _populated_ at boot. However, the folders themselves
  are not _created_ at boot, so _empty_ folders need to be copied.
- The _contents_ of /mnt and /media were excluded because they are
  places where certain external filesystems like USB Drives are mounted.
  However the empty folders need to be copied.
- /lost+found is filesystem-specific to ext3 and 4 and does not need to
  be copied.
- Quoting the exclude patterns will avoid expansion by shell.

Consider excluding unimportant subdirectories such as
`/home/*/.thumbnails/*`, `/home/*/.cache/*`, `/home/*/.local/share/Trash/*`

If GVFS is installed, `/home/*/.gvfs` must be excluded to prevent rsync
errors.
Note: gvfs has moved to /run/user/1000/gvfs
