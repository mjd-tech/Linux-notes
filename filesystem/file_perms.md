# File & Directory Permissions
owner, group, permissions (read, write, execute), SUID, SGID, Sticky Bit.

## owner, group
Every file on a Linux system, including directories, is "owned" by a
specific **user** and **group**. Access Permissions are defined separately for
user, group, and other.

- All users are members of a "primary group" (usually same name as username)
- By default, the user who creates a file or directory will become its owner.
- and the user's "primary group" becomes the group owner.
- "other" is a user who isn't the owner of the file and isn't a member of the group that owns the file.

## ls -l output

`ls -l` lists: file type; permissions; number of links; owner; group; size; modified date; filename

- The first field is 10-11 characters. The first character represents the file type.

    –   regular file
    b   block special file
    c   character special file
    d   directory
    l   symbolic link
    n   network file
    p   named pipe
    s   socket

- The next 3 characters are permissions for owner
- The next 3 characters are permissions for group
- The remaining 3 characters are permissions for other

    r   read
    w   write
    x   execute
    -   no permission
    
    s   (SUID or SGID), execute
    S   (SUID or SGID), no execute
    t   Sticky Bit, execute
    T   Sticky Bit, execute

- the 11th character will be a plus sign + if the file has an ACL (Access Control List)
- ACLs are rarely seen on desktop systems

## File vs Directory Permissions

| Permission | File                            | Directory                       |
|:-----------|:--------------------------------|:--------------------------------|
| read       | view file contents              | view contents, ie. ls command   |
| write      | modify file contents (edit)     | create or remove files from dir |
| execute    | execute a binary file or script | cd into directory               |
| SUID       | execute as file's owner         | no meaning                      |
| SGID       | no meaning                      | new files inherit group owner   |
| Sticky Bit | no meaning                      | only owner can delete files     |

### SUID
- For example: SUID is set for /usr/bin/passwd, to allow non-root users to change their own password.
- NEVER change SUID settings.

### SGID
- Most often seen on NAS systems, rather than Desktop systems
- NAS systems are intended for multiple users that can be put in groups, ie. sales, accounting, parents, kids
- You can create Directories that are only accessible by a specific group.

Example: sales directory. owner=root, group-owner=sales, and SGID set. Octal Permissions = 2770
- Only members of sales (and root) can access the directory
- All files/directories inherit the group-owner "sales"

### Sticky Bit
- only the file's owner can delete or rename a file or directory
- used by the /tmp directory, which is "world-writable"
- can be used in a NAS-style SGID directory, other users cannot delete your files.

## Symbolic vs Octal
- permissions are stored in 12 bits. (But see note below about gui file managers)
- can be represented by characters (symbolic), ie. `ls -l`
- can be represented by 3 or 4 Octal digits (Octal digits are 3 bits)

The bits are:
```
Special bits            user   group  other
----------------------  -----  -----  -----
SUID SGID "Sticky Bit"  r w x  r w x  r w x

```
### Display Octal Permissions
- octal "digits" represent 3 bits, so the range is 0-7
- `ls -l` does not display octal
- `stat somefile` shows both octal and symbolic (and other stuff)
- `stat -c '%n %a' somefile` shows only filename and octal
- `exa -l --octal-permissions somefile` shows both octal and symbolic. Note: exa usually not installed by default
- `exa -l --octal-permissions --no-permissions --somefile` shows only octal

#### Gui File Managers
Most gui file managers have an option to show octal permissions.
However you may see more than 4 octal digits. Example:
- files: 100644 
- directories: 40755 (it's actually 040775 but the leading 0 is not displayed)
Well, there are actually 16 bits involved. 
- The leftmost 4 bits represent the "type" of file. i.e. regular file or directory

On a Desktop Linux system 99% of the time,
- you are only concerned with the **Rightmost 3 digits** and you can **ignore** the rest.
- the 4th digit from the right will almost always be 0
- the 5th and 6th digit, represent whether it's a file or a directory, which you already know because the file manager shows directories as "folders" 

### Common Octal Permissions

```
Directories:
7   Full Access
5   Read-only
0   No Access

Files:
6   Full Access
4   Read-only
0   No Access

7   Full Access, executable
5   Read-only, executable
```

## Changing Permissions
- use the `chmod` command, accepts **symbolic** or **octal** permissions
- symbolic - can change one permission bit at a time, while leaving other bits alone.
- symbolic - best way to set/unset SGID and Sticky bit. (You should never need to mess with SUID)
- octal - don't use to set/unset SGID or Sticky bit, use symbolic
- octal - need to specify 3 digit permissions. cannot set/unset individual bits

## chmod Symbolic Permissions

Usage: chmod {options} filename

| Options | Definition        |
|---------|-------------------|
| u       | owner             |
| g       | group             |
| o       | other             |
| a       | all (same as ugo) |
| +       | add permission    |
| -       | remove permission |
| =       | set permission    |
| x       | execute           |
| X       | execute - only if it's a directory or already has execute set |
| w       | write             |
| r       | read              |
| s       | SUID/SGID         |
| t       | Sticky bit        |

### examples

```
# Remove all access to "other", leave "user" and "group" permission bits alone.
chmod o-rwx filename

# Add execute bit for "owner', leave "group"" and "other" permission bits alone
chmod u+x filename

# Add execute bit for everyone, leave the other permission bits alone
chmod a+x filename
chmod ugo+x filename
chmod +x filename

# Set all bits for a file: owner=read-write; group=read-only; other=read-only
chmod u=rw,go=r filename
chmod 644 filename

# Same, but for a directory, (directories need execute bit set)
chmod u=rwx,go=rx filename
chmod 755 dirname

# Set SGID bit on a directory
chmod g+s dirname

# Set Sticky bit on a directory
chmod +t dirname
```

### Recursive Permission Changes
- while chmod has a `-R` recursive option, you almost NEVER use that.
- Typically you want directories to have one set of permissions and files to have another.
- use `find`, like so:


    cd /path/to/someDirectory
    pwd   # make 100% sure you are where you need to be
    
    # Files
    find . -type f -exec chmod 644 {} +
    
    # Directories
    find . -type d -exec chmod 755 {} +

While either of these commands can be done by specifying the path directly,
and avoiding the `cd` and `pwd`, it is dangerous to do so, especially if
you are running as root.

**WARNING:** Recursively deleting, chown-ing, or chmod-ing can be dangerous.
You will not be the first, nor the last, person to add one too many spaces into the command.
This example will **hose your system**:

    # DON'T DO THIS
    sudo chmod 600 -R / home/john/Desktop/tempfiles

Note the space between the first / and home. This will apply the permissions to the entire file system,
and will be extremely painful to get it back to where it should be.

### Changing the File Owner and Group

use `sudo chown user:group filename`

```
# change owner to tux:
sudo chown tux somefile

# change group owner to penguins
sudo chgrp penguins somefile
# or
sudo chown :penguins somefile

# both at the same time
sudo chown tux:penguins somefile
```

### Recursive Ownership changes
- Ok to use the `-R` option with chown.

    cd /path/to/someDirectory
    pwd   # make 100% sure you are where you need to be
    sudo chown -R tux:penguins .

