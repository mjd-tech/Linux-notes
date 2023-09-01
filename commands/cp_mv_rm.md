# cp mv and rm
Copy/move/remove files and directories.

# cp
**Syntax:**

    cp [options] source [source2]... dest

If source is a directory, or there are multiple sources, then dest must be a directory.  
If you want to concatenate files, use cat

**Most useful options**

| option | description                                                             |
|--------|-------------------------------------------------------------------------|
| -a     | archive mode: recursive, preserve ownership, permissions and timestamps |
| -v     | verbose - print informative messages                                    |
| -r -R  | recursive copy                                                          |
| -u     | update - copy when source is newer than dest                            |
| -f     | force copy by removing the destination file if needed                   |
| -i     | interactive - ask before overwrite                                      |
| -l     | hard link files instead of copying                                      |
| -s     | make symbolic link instead of copying                                   |

**Examples:**

```
cp file.txt newfile.txt         # copy file to a new file
cp foo.txt{,.bak}               # backup foo.txt to foo.bak (uses Bash brace expansion)
cp file.txt target_dir          # copy file to another directory
cp foo.txt bar.txt target_dir   # multiple files to a directory
cp *.txt target_dir             # same, using Bash glob
cp -u *.txt target_dir          # "Update" mode - only copy newer files to target_dir
cp -t target_dir source         # target is first argument (useful for xargs | cp -t target_dir)

# copy directories (use -r)
cp -r source_dir target_dir
    # if target_dir exists, copy source_dir (and its contents) into target_dir
    # otherwise, create target_dir and copy only CONTENTS of source_dir, not the dir itself.

# Copy hidden files and directories:
cp -r .[a-z,A-Z,0-9]* target_dir
    # matches files starting with . and next character is a-z, A-Z, or 0-9
```

# mv
Rename SOURCE to DEST, or move SOURCE(s) to DIRECTORY.

```
mv source_file target_dir
mv file1.txt file.2.txt file3.txt target_dir
mv *.txt target_dir

# Rename
mv file target_file                 # same directory

mv file target_dir/target_file      # if target_file exists, it will be OVERWRITTEN without asking.

mv -i file target_dir               # ask before overwriting. answer with y/n
mv -n file target_dir               # don't overwrite (don't move file if it exists in target)

mv dir target_dir                   # if target_dir exists, dir becomes a sub-directory inside target_dir
```

# rm
Remove files and directories

NOTE: The `rm -rf` command is very dangerous and should be used with extreme caution!

```
rm filename                         # remove file
rm -f filename                      # force removal of write-protected file, don't prompt user 
rm -v filename                      # verbose, display information
rm filename1 filename2 filename3    # multiple files
rm *.png                            # multiple files, using Bash glob

# remove empty directory
rm -d dirname                       # rm -d is functionally identical to the rmdir command.

# remove non-empty directory, recursively
rm -r dirname
rm -rf dirname                      # force removal of write-protected files. BE CAREFUL WITH THIS

# Prompt Before each removal
rm -i filename1 filename2           # To confirm type y and press Enter:

# single prompt for the entire operation:
rm -I filename1 filename2 filename3 filename4
```
