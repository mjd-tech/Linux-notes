# Head and Tail
Displays first/last lines of a file or standard input, 10 lines by default

### Examples
```
# print first line only
head -n 1 /some/file

# print all lines except the last 3
head -n -3 /some/file

# Check if script file begins with shebang #! (checks first 2 characters)
[[ $(head -c2 file) = '#!' ]] && echo yes || echo no

# print first 2000 bytes of somefile.
head -c2000kB somefile

# print last 10 lines
tail example.txt

# Prints last 2 lines
tail -n2 example.txt

# Prints lines from the 2nd line. VERY USEFUL.
tail -n+2 example.txt

tail -c8 example.txt    # Prints the last 8 bytes from the file
tail -c+79 example.txt  # Prints the characters from the 79th byte.
tail -f logfile         # Prints last 10 lines and waits for new lines, 
                        # appending them to standard output.

# Concatenate multiple files, with filename headers. VERY USEFUL. Pick one.
head -n -0 foo.txt bar.txt baz.txt > all.txt
tail -n +1 foo.txt bar.txt baz.txt > all.txt
```

### Syntax
    head [-n count | -c bytes] [file ...]
    
    -n count    Display count number of lines instead of the default 10.
    -c bytes    Display count number of bytes, must provide a number.
    
    tail [options] [files]
    
    -n : Prints last N lines; +N prints lines from the Nth line in the file.
    -f : Prints the appended lines on the terminal as the file grows.
    -c : Prints the last N bytes of file; +N prints from the N byte in the file.
    
    Both:
    If more than a single file is specified, each file is preceded by a
    header consisting of the string ==> filename <==
    
    The number of bytes or lines can be followed by a multiplier suffix directly after the number.
    kB 1000, K 1024, MB 1000*1000, M 1024*1024, and so on.


