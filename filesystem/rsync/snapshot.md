# Snapshot Backups with rsync

- A Snapshot appears as a full file structure, identical to the "source"
  file structure.
- Restoring files is simply a matter of copying, using the command-line,
  or a gui file manager.

**Rotating Snapshots:**

- Snapshots are scheduled to run {hourly, daily, weekly, monthly, etc}
- A specified number of snapshots are kept, allowing you to go "back in
  time"
- Hard links are used on the "backup" partition to conserve disk space.
- The oldest snapshot is deleted when a new one is created.
- You need a script to do this, rsync does not have this built-in.

typical rsync command used in "rotating snapshots"

    rsync -a --delete --link-dest=/path/to/last-snapshot /path/to/source/ /path/to/new-snapshot

## Review of Hard Links

- Linux systems store files as *inodes*, which are numbers, not names.
- Inodes store file *attributes* (size, date, permissions, etc) and
  *pointers* to the actual file contents.
- Inodes do NOT store file *names* or *content*.
- A *directory* is a special inode. It has a list that maps (or "links")
  *names* to *inodes*.
- Each file has at least one "link".
- If you *copy* a file to a different directory, you create a new inode,
  and a copy of file contents, using more disk space.
- If you *hard link* a file to a different directory, you do NOT create
  a new inode, or use more disk space.
- When you delete a file, you actually "unlink" the inode from the
  directory.
- As long as an inode has at least one "link", the file still "exists".

This feature is used in rotating snapshots. Typically, relatively few
files change between snapshots. So you could have what looks like 10
separate "backups" of a file, but it is only consuming the disk space of
1.

## How To Date Stamp

We want a date stamp in the format of 2011-07-24 to represent July 24,
2011. This is known as an ISO-8601 compliant format.

    date -I
    # equivalent to
    date +%Y-%m-%d

To use this in a script, we set a variable:

    today=$(date -I)

We also need a variable equal to yesterday's date. We will let the date
command figure out what is "yesterday":

    yesterday=$(date -I -d "1 day ago")

## --link-dest to create hard links

- Rsync typically evaluates against the file structure in the target
  (destination) directory.
- If a file is different between the source and target, it gets copied.
  If it is the same, it does not.

In a "rotating snapshot" scenario, the "target" is going to be empty. We
need to evaluate against the *previous* backup, so we don't end up
copying the whole directory tree when only a few files have actually
changed.

The --link-dest option instructs rsync to evaluate against a different
file structure.

- Unchanged files are *hard linked* from the link-dest directory to the
  target directory. No new inodes created.
- Changed files get *copied* to target, new inodes created.
- If --deleted is used, deleted files (on the source) do not get copied
  or hard linked to the target directory. No new inodes created.

## Example Script

    #!/bin/sh
    # title:        web-backup
    # description:  Daily backup script for websites.
    # author:       mjd
    # date:         2012-05-20
    # version:      1.0
    # usage:        web-backup
    # notes:        rsyncs all websites to a backup directory.
    #               Puts files in date stamped directory. ie. 2012-05-20
    #               Keeps backups for x number of days (your choice).

    #---------------------------------
    # Modify these variables as needed
    #---------------------------------
    # directory that we want to back up. Include trailing slash.
    src=/var/www/

    # base directory where backups are stored. No trailing slash.
    base=/var/backups/websites

    # number of days to keep backups
    keep=7

    #---------------------------------------------------------------
    # ALL DONE WITH CONFIGURATIONS!
    # No real need to modify anything for the remainder of this file
    #---------------------------------------------------------------

    # Check the script is being run by root
    [ "$(id -u)" = "0" ] || { echo "$0 must run as root"; exit 1; }

    # Get today's date in ISO-8601 format YYYY-MM-DD
    today=$(date -I)

    # Get yesterday's date in ISO-8601 format YYYY-MM-DD
    yesterday=$(date -I -d "1 day ago")

    # Append today's date to base backup directory
    dest=$base/$today

    # Append yesterday's date to derive link directory
    lnk=$base/$yesterday

    # Create today's backup directory if it does not exist
    [ ! -d "$dest" ] && mkdir -p "$dest"

    # Create yesterday's backup directory if it does not exist
    [ ! -d "$lnk" ] && mkdir -p "$lnk"

    # Backup the source directory
    rsync -avh --delete --link-dest="$lnk" "$src" "$dest"

    # If rsync failed, exit now, do not remove oldest backup directory
    [ $? -ne 0 ] && {
        echo "rsync from $src to $dst failed"
        exit 1
    }

    # Remove oldest backup directory
    keep=$(( $keep + 1 ))
    oldest=$base/$(date -I -d "$keep days ago")
    [ ! -d "$oldest" ] && exit 0
    rm -r "$oldest"
