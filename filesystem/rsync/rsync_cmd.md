# rsync command

SYNOPSIS

    Local:  rsync [OPTION...] SRC... [DEST]

    Access via remote shell:
    Pull:   rsync [OPTION...] [USER@]HOST:SRC... [DEST]
    Push:   rsync [OPTION...] SRC... [USER@]HOST:DEST

    Access via rsync daemon:
    Pull:   rsync [OPTION...] [USER@]HOST::SRC... [DEST]
            rsync [OPTION...] rsync://[USER@]HOST[:PORT]/SRC... [DEST]
    Push:   rsync [OPTION...] SRC... [USER@]HOST::DEST
            rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/DEST

    Usages with just one SRC arg and no DEST arg will list the source files
    instead of copying.

## Options

rsync has many many options. These are the most common:

|                      |                                                                                               |
|:---------------------|:----------------------------------------------------------------------------------------------|
| -a --archive         | you almost always want this. recursive; preserve owner and permissions, symlinks, time stamps |
| -v --verbose         | gives information about what files are being transferred and a brief summary at the end       |
| -vv                  | more verbose, also gives info on what files are being skipped                                 |
| -z --compress        | compress data during transmission at the expense of higher cpu overhead                       |
| -n --dry-run         | perform a trial run with no changes made                                                      |
| -x --one-file-system | don't cross filesystem boundaries, don't backup things "mounted" into your source             |
| --delete             | delete files on destination that are not in the source. :!: *Be careful with this!*           |
| --progress           | give a rough idea of the percentage transferred and time remaining                            |
| --partial            | keep partially transferred files, useful for large files on a flaky link                      |
| -P                   | shortcut for --partial --progress                                                             |
| -A --acls            | preserve ACLS (Access Control Lists)                                                          |
| -X --xattrs          | preserve "extended file attributes"                                                           |
| -H                   | preserve hard links                                                                           |
| -h --human-readable  | output numbers with commas and units (K,M,G,T). Useful with --progress                        |
| --exclude=           | specify a filter rule                                                                         |
| --exclude-from=      | specify a file containing filter rules                                                        |
| --delete-excluded    | delete files on destination that are excluded in the source                                   |
| -e --rsh             | specify remote shell to use, most often used with ssh                                         |
| --rsync-path         | specify the rsync to run on remote host. Often used with sudo                                 |
| --link-dest          | used in "incremental snapshots". makes hardlinks to reference directory.                      |

## Trailing slash in the source path

This is only significant when using --recursive (implied by -a)

- No trailing slash: copy the directory itself, plus files and
  subdirectories within
- Trailing slash: copy only the files and subdirectories within
- A trailing slash on the *destination* has no significance, but is
  customary to include it
- The **destination** given on the command line **must exist**

For example: the directory *dir* contains *file*

    rsync -av /source_path/dir/ /dest_path/      # Result: /dest_path/file
    rsync -av /source_path/dir /dest_path/       # Result: /dest_path/dir/file  rsync creates /dest_path/dir/
    rsync -av /source_path/dir/ /dest_path/dir/  # Result: /dest_path/dir/file  only if /dest_path/dir/ exists

More examples:

    1) rsync -avz /src/foo  /dest      => ok. creates /dest/foo
    2) rsync -avz /src/foo/ /dest/foo  => ok 
    3) rsync -avz /src/foo/ /dest/foo/ => ok. trailing slash on DESTINATION has no significance
    4) rsync -avz /src/foo/ /dest      => dangerous!!! overwrite dest content
           solution: remove trailing / at /src/foo/
    5) rsync -avz /src/foo /dest/foo   => not ok, creates /dest/foo/foo
           solution: add trailing / to source

## Progress

Show cumulative progress

    rsync -av --info=progress2 /path/to/src /path/to/dst

## Filter rules

Normally, you're going to use "recursion" (implied by -a). This means
copy the source directory and all sub-directories under it.

- Often, you want to exclude certain files or directories, such as cache
  directories, temp directories or just stuff you don't need to be
  copied.
- For this, you need filter rules.
- There are two types, include and exclude.
- Most of the time you use exclude filter rules.

### Exclude Filter Quick Reference

Rules can be specified on the command line:

    rsync -avx --exclude=somefile.txt SRC DEST  # You can use multiple --exclude= statements.

Or you can put your excludes in a file.

    rsync -avx --exclude-from=my_excludes.txt SRC DEST

Excludes files contain one filter rule per line, can contain comment
lines (begin line with \#), blank lines ignored.

Filter Rules use "patterns" similar to shell wildcards.

| Pattern      | Description                                                                       |
|:-------------|:----------------------------------------------------------------------------------|
| foo          | file or directory named **exactly** foo                                           |
| /foo         | file or directory named exactly foo, "anchored" at the "root of the transfer".    |
| foo\*        | file or directory that **begins with** foo: foo, foobar, foobar.baz, etc.         |
| foo/         | directory named exactly foo, but not a file                                       |
| foo/\*       | files and sub-directories within foo                                              |
| foo/\*/baz   | baz, exactly one subdirectory under foo/ i.e. foo/bar/baz but not foo/bar/bif/baz |
| foo/\*\*baz  | baz, zero or more subdirectories under foo/                                       |
| foo/\*\*/baz | baz, one or more subdirectories under foo/                                        |

Example:

    rsync -avx --exclude=foo /home/user/ /backup/dir/

This will sync everything except:

    /home/user/foo                (file, relative to the "root of the transfer")
    /home/user/foo/               (or a directory - neither the directory or its contents get synced)
    /home/user/Documents/July/foo (any file named foo, anywhere in the path) 
    /home/user/Documents/foo/bar  (any directory named foo, anywhere in the path. The bar file will not be synced because the foo diretory is not synced) 

### Filter Details

- Selection of which files to transfer (include) and which files to skip
  (exclude).
- Can specify on command line or read them from a file.

How they work:

- rsync checks each file or directory name to be transferred against the
  list of include/exclude patterns
- starts with the first pattern in the list and continues to the last,
  in the order you specify.
- the first matching pattern is acted on:
  - if exclude pattern, name is skipped;
  - if include pattern, name is not skipped;
- if no matching pattern is found, name is not skipped.

Filter rules are specified on the command line with

- --exclude
- --include
- --filter (or -f)

If using --filter or -f, you can use the words *include* or *exclude*
but more common is to use + for include and - for exclude.

Filter rules may also be put in a file.  
Put --exclude-from=*filename* or --include-from=*filename*  
The file can contain both include and exclude rules.  
So it is common to just put all rules in one file and call it with
--exclude-from=some_exclude_file

A common use is to list *include* rules first, then as the last rule,
exclude \* (everything), but be careful when using --recursive (see
Patterns below)

### Patterns

All patterns are *relative* to the source path given on the command
line.

Patterns can contain the following wildcard characters:

| Wildcard | Description                                                                     |
|:---------|:--------------------------------------------------------------------------------|
| \*       | matches any path component, except slash /                                      |
| \*\*     | matches any path component, including slashes                                   |
| ?        | matches any single character, except slash /                                    |
| \[       | introduces a character class, such as \[a-z\] or \[\[:alpha:\]\]                |
| \\       | escape a wildcard character. Two backslashes are needed for a literal backslash |

Note:

- Things get tricky when using the --recursive (-r) option (which is
  implied by -a)
- rsync "visits" each directory and applies the filter rules, on each
  "visit"
- To include a **file** "/foo/bar/baz.txt". "/foo" and "/foo/bar" must
  not be excluded
- This is particularly important when using a trailing \* rule.

For instance, **this won't work:**

    + /foo/bar/baz.txt
    + /bif.txt
    - *

baz.txt **will not** be synced, because its parent directories are
excluded by the the "- \*" rule.  
bif.txt **will** be synced. It's in the "root of the transfer". The
"root of the transfer" always gets "visited" by rsync.  
One solution is to add specific include rules for all the **parent**
directories that need to be visited.

    + /foo/
    + /foo/bar/
    + /foo/bar/baz.txt
    - *

Another solution is to ask for **all** directories in the hierarchy to
be included by using a single rule:

    + */
    + /foo/bar/baz.txt
    - *

Note, these solutions will create a bunch of empty directories in the
destination.

The following modifiers are accepted after a "+" or "-":

|     |                                                                                                                                                                                |
|-----|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| /   | match the absolute pathname of the current item. For example, "-/ /etc/passwd" would exclude the passwd file any time the transfer was sending files from the "/etc" directory |
| !   | the "not" operator. the include/exclude should take effect if the pattern fails to match. For instance, "-! \*/" would exclude all non-directories                             |
| C   | is used to indicate that all the global CVS-exclude rules should be inserted as excludes in place of the "-C". No arg should follow                                            |
| s   | the rule applies only to the sending side. The default is for a rule to affect both sides                                                                                      |
| r   | the rule applies only to the receiving side                                                                                                                                    |
| p   | indicates that a rule is perishable, meaning that it is ignored in directories that are being deleted                                                                          |

## Backup entire system

Must be run as root.  
<https://wiki.archlinux.org/title/Rsync#Full_system_backup>

    # transfer a copy of your "/" tree, excluding a few select folders.
    # This command depends on brace expansion available in both the bash and zsh shells.
    rsync -aAXHv --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} / /path/to/backup/

This command uses Bash "brace expansion" to auto-generate multiple
`--exclude=` expressions.  
If you are using /bin/sh, you cannot use brace expansion so you must use
explicit multiple --exclude statements. i.e.

    rsync -aAXHv --exclude="/dev/*" --exclude="/proc/*" --exclude="/sys/*" ...

or use a file to store the excludes and use --exclude-from=some_file.

- Using the -aAHX set of options: archive mode; preserve ACLs, preserve
  extended attributes, preserve hard links.
- The `--exclude` option will cause files that match the given patterns
  to be excluded.
- The *contents* of /dev, /proc, /sys, /tmp and /run were excluded
  because they are *populated* at boot. However, the folders themselves
  are not *created* at boot, so *empty* folders need to be copied. The
- /lost+found is filesystem-specific to ext3 and 4 and does not need to
  be copied.
- Quoting the exclude patterns will avoid expansion by shell.

## Using rsync over ssh

- rsync needs to be installed on both the local and remote hosts.
- You need to have a user account on the remote host.
- If you intend to do this frequently, set up "passwordless" ssh
- Specify remoteuser@remotehost:/path/on/remotehost as either *source*
  or *destination*
- You can use the -z (compress) option, but there's no benefit if
  sending already compressed files like mp3, jpg, mpeg, etc

<!-- -->

    rsync -avz /home/localuser/Documents remoteuser@remotehost:/home/remoteuser/backups/

Since there is no trailing slash on the *source*, a directory named
Documents will be created (if not already there) on the remote host. If
there is already a Documents directory on the remote host, you can also
do this:

If you want to delete any files on the remote host that are no longer on
the source host, use the *--del* or *--delete* argument.

    rsync -avz --del /home/localuser/Documents remoteuser@remotehost:/home/remoteuser/backups/

## Rsync multiple sources with one command

You can rsync multiple files or directories using a single rsync line.
This is useful when coping over the network as you only need to enter
your ssh password once. Here we will copy the source directories /data1
and /data2 to the remote machine "somemachine" in the /backups/
directory.

    rsync -avz /data1 /data2 calomel@somemachine:/backups/

The final path specified is the destination.

## Rsync pull instead of push

You can also pull the data from a remote machine to your local box.

    rsync -avz remoteuser@remotehost:/sourcedir/ /local_destination_dir/

If your remote host user account is non-root and you want to rsync
root-owned files:  
On the remote host, edit `/etc/sudoers` (the "sudo" package must be
installed) and add this:

    # Allow admin user to run rsync as root, without password prompt.
    remoteuser ALL=(root)NOPASSWD:/usr/bin/rsync

On the local host, construct the rsync command similar to this:

    rsync -avz --del --rsync-path="sudo rsync" remoteuser@remotehost:/sourcedir/ /local_destination_dir/

## Pull multiple files with rsync

Here we are pulling multiple files from a remote server to our local
machine. We have to use curly braces to tell rsync we want only the
files "file1", "file2" and "file3." Using this method you only have to
enter your ssh password once to collect all the files.

    rsync -avz calomel@somemachine:/remote_machine/{file1,file2,file3} /local/disk/

## Rsync when remote files or directories contain spaces

Spaces cause all sorts of problems.

- When using rsync over ssh, part of the command line is parsed twice,
  once by the shell on the local host, once by remote host's shell.
- Shells consider spaces to be delimiters between "words". Shells will
  "word-split" a command line before executing.
- A filename with spaces will be considered multiple "words". It should
  be just one "word".
- Shells deal with this by "escaping" the spaces with either single
  quotes, double quotes or backslashes.
- With remote rsync, you have to "double escape" filenames that have
  spaces.
- So, use single quotes to encompass the entire filename, then escape
  the spaces with backslashes.

The following line shows the remote directory name "/I Hate Spaces/" and
the file name "some File.avi", both contain spaces. We will be rsync'ing
the file to our local directory "/current_dir/".

    rsync -av user@remotehost:'/I\ Hate\ Spaces/some\ File.avi' /current_dir/

When this command is parsed by the local host, the shell splits it into
"words", based on "whitespace" and removes quotes.

|          |                                                    |                                |
|----------|----------------------------------------------------|--------------------------------|
| 1st word | rsync                                              | the command to execute         |
| 2nd word | -av                                                | options for rsync              |
| 3rd word | user@remotehost:/I\\ Hate\\ Spaces/some\\ File.avi | the "source" argument to rsync |
| 4th word | /current_dir/                                      | the "destination" argument     |

notice the local shell has removed the single quotes, but has not split
the filename into separate words. It has not removed the backslashes.
These will be removed by the remote host.

rsync will split the 3rd word above into two, based on the colon.

|          |                                    |                                                                             |
|----------|------------------------------------|-----------------------------------------------------------------------------|
| 1st half | user@remotehost                    | instructs the localhost to run ssh and connect to the remote host as "user" |
| 2nd half | /I\\ Hate\\ Spaces/some\\ File.avi | gets passed to the remote shell, where it will be parsed                    |

The remote shell removes the backslashes, leaving the filename intact,
with its spaces, as one "word".

## Execute remote shell command to rsync files

It is important to note rsync can also execute commands on the remote
machine to help you generate a list of files copy. The shell command is
expanded by your remote shell before rsync is called. The following line
will run the find command on the remote machine in the video directory
and rsync all "avi" files it finds to our machine in the /download
directory.

    rsync -avR calomel@somemachine:'`find /data/video -name "*.[avi]"`' /download/

## Pull data from a remote machine to local server using ssh

The following command will pull the data from "remote_machine" in
/stuff/data/ and place it on the local system in
/BACKUP/remote_machine/. The arguments "-avx" will set archive mode (-a)
equivalent to -rlptgoD, be verbose (-v) and will not cross file system
boundaries (-x) like NFS or samba. The timeout command makes sure rsync
will not hang if the remote system is unreachable after 30 seconds. We
will be deleting any files on the target directory
(/BACKUP/remote_machine/) that are \_not\_ found in the source directory
(data/). If you do not want to allow rsync to delete any files then take
out "--delete-excluded". The directory structure of the target machine
will look like /BACKUP/remote_machine/data/ and this is considered a
non-relative path option. Notice /stuff/ is not in the path.

    rsync -avx --timeout=30 --delete-excluded backupuser@remote_machine:/stuff/data/ /BACKUP/remote_machine/

If you wanted the target directory structure to be relative you can add
the argument "-R". The directory structure would then look like
/BACKUP/remote_machine/stuff/data/ as the sync path name starts / on the
source machine. The command with "-R" added looks like:

    rsync -Ravx --timeout=30 --delete-excluded backupuser@remote_machine:/stuff/data/ /BACKUP/remote_machine/

## Gotchas

      Be careful with the --delete command. 
      If your using a source dir that is not mounted (nfs,cifs,etc) but the mount for the dir is still there then you will sync your blank dir.
      All remote files will be deleted.
      
      Be careful with slashes after the dir names on the source. 
      A slash after the dir name compared to no slash after the dir name will do 2 totally different things.
      
      Running out of memory with the older version of Rsync when doing a huge amount of files. 
      Use the newer version due to its ability to do incremental file lists.

## How rsync determines which files are "outdated"

rsync has three ways to decide if a file is outdated:

    -Compare the **size** of source and destination.
    -Compare the **timestamp** of source and destination.
    -Compare the static **checksum** of source and destination.

These checks are performed before transferring data. Notably, this means
the static checksum is distinct from the stream checksum - the later is
computed while transferring data.

By default, rsync uses only 1 and 2. Both 1 and 2 can be acquired
together by a single stat, whereas 3 requires reading the entire file
(this is independent from reading the file for transfer). Assuming only
one modifier is specified, that means the following:

    By using --size-only, only 1 is performed - timestamps and checksum are ignored. A file is copied unless its size is identical on both ends.
    By using --ignore-times, neither of 1, 2 or 3 is performed. A file is always copied.
    By using --checksum, 3 is used in addition to 1, but 2 is not performed. A file is copied unless size and checksum match. The checksum is only computed if size matches.
