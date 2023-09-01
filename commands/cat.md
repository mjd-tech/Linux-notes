# cat
display the contents of one or more files. Cat is short for concatenate (combine), 
but is most often used to quickly view one small file. For larger files, use *less*

```
# View a file
cat /path/to/myfile

# Display line numbers
cat -n /path/to/myfile

# Show non-printing characters except TAB
cat -v /path/to/myfile

# Display a $ at end of each line, useful to find trailing whitespace
cat -E /path/to/myfile

# Show TAB characters as ^I
cat -T /path/to/myfile

# Show-all, equivalent to -vET
cat -A /path/to/myfile

# Combine several files into one file, the "intended" use of cat
cat file1 file2 "file 3" > newfile

# Squeeze multiple consecutive blank lines into one blank line
cat -s file1
```
