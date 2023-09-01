# df and du
disk free and disk used.

## df
Disk Free. Report used and available space on one or more devices.

Most useful:

    df -h           # all filesystems in "human-readable" format
    df -h --total   # as above, plus a grand total

If you think you are running out of inodes:

    df -ih

## du
Disk Used. Estimate file space usage.

du will report usage for each directory and subdirectory.

Examples:

    du -h              # current directory, recursive, "human-readable" format
    du -h /home        # /home directory, recursive.
    du -hs /home       # summarize, don't print each directory, just the grand total
    du -h -d1 /home    # depth = 1. just print total for /home/user1, /home/user2, etc. then the grand total. 

Unless you use -s (summarize), or -d (depth) du will print a line for
each subdirectory, this can be a very large number. The -s option is
equivalent to -d0 (zero depth), so you cannot use both -s and -d.
