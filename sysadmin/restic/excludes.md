# Excludes file

- Save this in ~/restic/excludes.txt
- Modify to your needs.
- When in doubt, use the /full/path

```
# excludes for restic backup.
# see bottom of this file for notes and examples

# We want to backup the following directories:
# /boot /etc /home /opt /root /srv /usr/local /var
# Rather than specifying these to "restic backup" we specify / (everything)
# And exclude what we don't want

# Exclude these top level directories
/bin
/dev
/lib
/lib64
/media
/mnt
/proc
/run
/sbin
/sys
/tmp

# We want /usr/local but not anything else under /usr
/usr/*
!/usr/local

# this is specific to ext4 file systems
lost+found

# Exclude PID files and Lock files
/var/run/*
/var/lock/*


# Stuff in home directories
/home/*/.cache/*
/home/*/.local/share/gvfs-metadata/*
/home/*/.local/share/Trash/*
/home/*/Downloads/*
/home/*/Videos/*
/home/*/Music/*

# Clutter files from Windows and Mac
Thumbs.db
.DS_Store

# Trash
.Trash-1*

# NodeJS modules, npm cache
node-modules
.npm

# NOTE:
# Patterns are tested against the FULL PATH, even if restic is passed a relative path to save.
# A leading slash / anchors the pattern at the root directory.
# Trailing slash / is ignored.
# This means, /bin matches /bin/* but does not match /usr/bin/*

# Examples:
# foo       exclude any file or directory named exactly foo
# foo*      exclude any file or directory that begins with foo
# /foo      excludes /foo and everything within /foo
# /foo/*    includes /foo but nothing within /foo - in the backup, /foo is emtpy directory.
# !/foo/bar includes /foo/bar even though we excluded /foo/* above
```
