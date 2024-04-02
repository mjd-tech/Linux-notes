# rsync command
- copy directories/files to/from local/remote location.
- delta-transfer algorithm sends only the differences between the source files and destination.
- widely used for backups and mirroring.
- rsync **must** be installed on both source and destination hosts

> ⚠️ **WARNING**  
> Always test rsync with `--dry-run` or `-n`  
> This shows what **would** happen, without actually doing anything

## Syntax

    Local:  rsync [OPTION...] SRC... [DEST]

    Via ssh:
    Pull:   rsync [OPTION...] [USER@]HOST:SRC... [DEST]
    Push:   rsync [OPTION...] SRC... [USER@]HOST:DEST

    Via rsync daemon:
    Pull:   rsync [OPTION...] [USER@]HOST::SRC... [DEST]
            rsync [OPTION...] rsync://[USER@]HOST[:PORT]/SRC... [DEST]
    Push:   rsync [OPTION...] SRC... [USER@]HOST::DEST
            rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/DEST

    Usages with just one SRC arg and no DEST arg will list the source files
    instead of copying.

## Options

rsync has many many options. These are the most common:

| Option               | Description                                                              |
| :------------------- | :----------------------------------------------------------------------- |
| -a --archive         | recursive; preserve owner and permissions, symlinks, time stamps         |
| -v --verbose         | list files transferred, and brief summary at the end                     |
| -vv                  | more verbose, also gives info on what files are being skipped            |
| -z --compress        | compress data during transmission at the expense of higher cpu overhead  |
| -n --dry-run         | perform a trial run with no changes made                                 |
| -x --one-file-system | don't backup things "mounted" into your source                           |
| --delete             | delete files on destination that are not in the source.                  |
| --progress           | give a rough idea of the percentage transferred and time remaining       |
| --partial            | keep partially transferred files, useful for large files on a flaky link |
| -P                   | shortcut for --partial --progress                                        |
| -A --acls            | preserve ACLS (Access Control Lists)                                     |
| -X --xattrs          | preserve "extended file attributes"                                      |
| -H                   | preserve hard links                                                      |
| -h --human-readable  | output numbers with commas and units (K,M,G,T). Useful with --progress   |
| --exclude=           | specify a exclude filter rule                                            |
| --exclude-from=      | specify a file containing filter rules                                   |
| --delete-excluded    | delete files on destination that are excluded in the source              |
| -e --rsh             | specify remote shell to use, most often used with ssh                    |
| --rsync-path         | specify the rsync to run on remote host. Often used with sudo            |
| --link-dest          | used in "incremental snapshots". makes hardlinks to reference directory. |

Note: You can use the -z (compress) option, but there's no benefit if
sending already compressed files like mp3, jpg, mpeg, etc

## Trailing slash in the source path

This is only significant when using --recursive (implied by -a)

- No trailing slash: copy the directory itself, plus files and subdirectories within
- Trailing slash: copy only the files and subdirectories within
- A trailing slash on the *destination* has no significance, but is customary to include it
- The **destination** given on the command line **must exist**

Examples:
```bash
rsync -avz /src/foo  /dest      => ok. creates /dest/foo
rsync -avz /src/foo/ /dest/foo  => ok 
rsync -avz /src/foo/ /dest/foo/ => ok. trailing slash on DESTINATION has no significance
rsync -avz /src/foo/ /dest      => dangerous!!! overwrite dest content
    solution: remove trailing / at /src/foo/
rsync -avz /src/foo /dest/foo   => not ok, creates /dest/foo/foo
    solution: add trailing / to source
```

## Progress
Show cumulative progress
```bash
rsync -av --info=progress2 /path/to/src /path/to/dst
```

## Using rsync over ssh
- rsync needs to be installed on both the local and remote hosts.
- You need to have a user account on the remote host.
- Specify `remoteuser@remotehost:/path/on/remotehost` as either _source_ or _destination_

If you intend to do this frequently, for example a daily cron job
- set up "shared key authentication" (passwordless) ssh to the remote host
- put the remote host in your ~/.ssh/config file
- Specify `remotehost:/path/on/remotehost` as either _source_ or _destination_

## Dealing with root owned files
The obvious method is to do everything as root.
- set up ssh "shared key authentication" for root
- put the remote host in /root/.ssh/config
- run the rsync job as root.
- if it's a scheduled task, put in root's crontab

But now you have to maintain `.shh/` stuff and crontabs for root,
on both the source and destination hosts.

For hosts on a home network, you'd rather do everything as your user.

First, allow `sudo rsync` without password, on **both** the source and destination host:
```bash
sudo visudo -f /etc/sudoers.d/myconf
# Put this:
%sudo ALL = NOPASSWD: /usr/bin/rsync
```
Next, construct your rsync command something like this:
```bash
# Push
sudo rsync -av -e "sudo -u $USER ssh" --rsync-path="sudo rsync" /path/to/source remotehost:/path/to/destination/

# Pull
sudo rsync -av -e "sudo -u $USER ssh" --rsync-path="sudo rsync"  remotehost:/path/to/source/ /path/to/destination
```
This command does the following.
- the first `sudo rsync` runs local rsync process as root
- `-e "sudo -u $USER ssh"` runs the ssh process as your user, with your user's `~./ssh` stuff
- `--rsync-path="sudo rsync"` runs rsync on the remotehost as root.

In other words, rsync runs as root, but uses the user's ssh client to make the connection. No passwords needed, so can run unattended.

## Delete files on destination
To delete files in the destination that are no longer in the source,  
use the `--delete` argument.

```bash
# ALWAYS test --delete with --dry-run
rsync --dry-run -av --delete /path/to/source/ remotehost:/path/to/destination
# omit --dry-run once you're sure it's correct
```
> ⚠️ **WARNING**  
> Be careful when using USB drives as either source or destination  
> If a USB drive is not mounted, **BAD THINGS** can happen

This issue is especially critical with **cron jobs**.  
You might remove a USB drive, and forget that you have a daily cron job
that rsyncs to/from that drive.

Before running rsync, make sure USB drives are actually mounted.  
Put something like this in your rsync script:
```bash
#!/bin/bash

# Check if local USB drive is mounted
mountpoint -q /path/to/usb/mount || exit

# Check if remote USB drive is mounted
ssh remotehost mountpoint -q /path/to/usb/mount || exit

# Put your rsync command(s) below here
```
## Remote source or destination directory contain spaces
Spaces cause problems:
```bash
# This will not work, although it looks like it's quoted correctly:
rsync -av remotehost:"/path/to/my source/" "/path/to/my destination/"
```
In this case `"/path/to/my destination/"` is fine.  
But `"/path/to/my source/"` is parsed twice:
1. by the shell on the local host
2. by the shell on the remote host

This results in:
- The local shell removes quotes before passing the path to the remote shell
- The remote shell "sees" an unquoted path and rsync fails.

The easiest way to deal with this put extra quotes on the remotehost path.

```bash
# Remote path: double quotes inside single quotes
rsync -av remotehost:'"/path/to/my source/"' "/path/to/my destination/"
```
The local shell removes the single quotes and passes the double-quoted path to the remote shell.

## Use a command to generate list of files to copy
This trick uses rsync's `--files-from=` argument, which expects a filename,  
but we use Bash "process substitution".  
This runs a command and automatically puts the output into a temp file that rsync can use.

For example:
rsync all .avi files in the myVideos directory.

```bash
# Push to remote host
rsync --dry-run -av --files-from=<(find /myVideos -iname "*.avi" -printf "%P\n") remotehost:/path/to/destination

# Pull from remote host
rsync --dry-run -av --files-from=<(ssh remotehost 'find /myVideos -iname "*.avi" -printf "%P\n"') remotehost:/myVideos /path/to/destination
```
- Note that we have to wrap the find command in single quotes when it's executed on the **remotehost**.
- The `-printf "%P\n"` option prints the relative path, not the default full path, and inserts a newline after each file that it finds.
- rsync `--files-from` expects a newline delimited list of files.

## Gotchas

**Be careful with the `--delete` command.**

> If you're using a source dir that is not mounted (nfs,cifs,etc),  
> but the mount dir is still there, then you will sync your blank dir.  
> All destination files will be deleted.
>
> Make sure the source directory is actually mounted before using rsync

**Be careful with slashes after the dir names on the source.**

A slash after the dir name compared to no slash after the dir name will do 2 totally different things.

**Quoting Issues**

- Anything involving remote hosts is subject to quoting issues.
- The remote part of the command line is parsed twice.
- Most problems are solved with the "double quotes inside single quotes" method.
- You sometimes need a combination of single quotes, double quotes, and backslash escapes for the remote stuff.

## How rsync determines which files are "outdated"

1. Compare the **size** of source and destination.
2. Compare the **timestamp** of source and destination.
3. Compare the static **checksum** of source and destination.

These checks are performed before transferring data.

By default, rsync uses only 1 and 2

- By using `--size-only`, only 1 is performed - timestamps and checksum are ignored. A file is copied unless its size is identical on both ends.
- By using `--ignore-times`, neither of 1, 2 or 3 is performed. A file is always copied.
- By using `--checksum`, 1 and 3 are performed. A file is copied unless size and checksum match. The checksum is only computed if size matches.