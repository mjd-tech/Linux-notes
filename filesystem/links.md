# Hard links vs Symbolic links

For review, a "file" consists of:

1.  Directory listing: the file name and its inode
2.  Inode (index node): the metadata for the file - owner, permissions,
    timestamps and physical location on the device
3.  The actual Data

## Hard Links

In Linux when you perform an listing in a directory the listing is
actually a list of references that map a filename to an inode. When you
create a hard link, that hard link is yet another reference to the
**same** inode as the original file.

Hard links have some limitations however, in most (but not all)
Unix/Linux distributions hard links cannot be created for directories.
Hard links are also not allowed to cross file systems.

## Symbolic Links (aka Symlinks or soft links)

A symbolic link is similar to a hard link in that it is used to link to
an already existing file, however it is very different in its
implementation. A symbolic link is not a reference to an inode but
rather an pointer that redirects to another file or directory. When you
create a symlink, you create a **new** inode.

Symlinks do not have the same restrictions as hardlinks, a symlink can
be used to reference a file in another file system. Symlinks can also be
used to point to a directory.

## When to use a Symbolic link or a Hard Link

There are many cases when you would want to use a symlink and really
only a few where you would want to use a hard link.

One case where hard links are beneficial is when doing "snapshot"
backups using rsync or similar tool. If you want to keep several
backups, with the directory tree intact, but don't want to duplicate
files that are the same, use hard links.

      * a file can be in two or more places without having to duplicate data on disk
      * Each Hard Link does not increase the number of inodes, there is a limit to the number of inodes on a filesystem.

## Creating a Symbolic Link

Now that we have covered when and why to use hard links and symlinks
lets cover how to create them.

### Symlinking a Directory

Creating a symlink is easy, simply run ln -s and specify the source
(directory you want to symlink to) and the linkname (the symlinks name).

Format:

    ln -s <source> <linkname>

In the below example I will be creating a symlink named dir2 that links
to dir1.

Example:

    ln -s dir1 dir2

    ls -l
    total 4
    drwxrwxr-x 2 madflojo madflojo 4096 Oct 8 06:38 dir1
    lrwxrwxrwx 1 madflojo madflojo 4 Oct 8 06:38 dir2 -> dir1

As you can see from the output of the ls command the link looks very
different from the directory. The format linkname -\> target is how all
symlinks show, whether they are linking to a file or a directory. You
may also notice that the file type shows as l for symbolic link.

### Symlinking a File

Creating a symlink for a file is the same process as creating a symlink
for a directory.

    cat file.real 
    This is file.real

    ln -s file.real file.sym

    ls -l
    total 4
    -rw-rw-r-- 1 madflojo madflojo 18 Oct 9 18:49 file.real
    lrwxrwxrwx 1 madflojo madflojo 9 Oct 9 18:49 file.sym -> file.real

    cat file.sym
    This is file.real

As you can see the symlink for a file appears the same as a symlink for
a directory. This is due to the fact that a symbolic link is the same
whether it points to a file or a directory.

## Creating Hard Links

Creating a hard link can be done via the ln command as well. With hard
links however we will not be using the -s flag.

Format:

    ln <source> <linkname>

Example:

    ln file.real file.hard

    ls -l
    total 8
    -rw-rw-r-- 2 madflojo madflojo 18 Oct 9 18:49 file.hard
    -rw-rw-r-- 2 madflojo madflojo 18 Oct 9 18:49 file.real
    lrwxrwxrwx 1 madflojo madflojo 9 Oct 9 18:49 file.sym -> file.real

    cat file.hard
    This is file.real

As you can see the hard link will also allow you to link to the contents
of file.real but the output of ls is very different. As far as the file
system is concerned there is no difference between file.real and
file.hard as they are both references to the same inode (10097087).

The only way to even show that file.hard is a hard link is to look at
the links count in the inode. You can do this with the stat command.

    stat file.hard 
    File: file.hard
    Size: 18 Blocks: 8 IO Block: 4096 regular file
    Device: fc00h/64512d Inode: 10097087 Links: 2
    Access: (0664/-rw-rw-r--) Uid: ( 1001/madflojo) Gid: ( 1001/madflojo)
    Access: 2013-10-09 18:56:36.079158024 -0700
    Modify: 2013-10-09 18:49:46.477126917 -0700
    Change: 2013-10-09 18:56:29.903127399 -0700
    Birth: -

While this shows that the inode for file.hard has 2 links it does not
necessarily show which is the original file and which is the hard link.

Since a hard link is simply a reference to an inode, you can actually
delete the original file and the data will still be available via the
hard link. Where as with a symlink if the target is deleted the symlink
will no longer be able to provide data.

    rm file.real

    ls -l
    total 4
    -rw-rw-r-- 1 madflojo madflojo 18 Oct 9 18:49 file.hard
    lrwxrwxrwx 1 madflojo madflojo 9 Oct 9 18:49 file.sym -> file.real

    cat file.hard 
    This is file.real

    cat file.sym
    cat: file.sym: No such file or directory

The symlink is now a "broken" link where as the hard link is now
indistinguishable from any other file.
