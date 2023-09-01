# Deleting Multiple Files

If you're trying to delete a very large number of files at one time, you
will probably run into this error:

    /bin/rm: Argument list too long.

The problem is that when you type something like `rm -rf *`, the `*` is
replaced with a list of every matching file, like "rm -rf file1 file2
file3 file4" and so on. There is a reletively small buffer of memory
allocated to storing this list of arguments and if it is filled up, the
shell will not execute the program.

To get around this problem, use the find command with its built-in
"-delete" option:

    find . -type f -delete

You can also show the filenames as you’re deleting them:

    find . -type f -print -delete

or even show how many files will be deleted, then time how long it takes
to delete them:

    ls -1 | wc -l && time find . -type f -delete
    100000
    real    0m3.660s
    user    0m0.036s
    sys     0m0.552s
