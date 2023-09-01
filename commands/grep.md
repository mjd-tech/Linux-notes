# Grep
Search for regex PATTERN in FILE or standard input.

## Examples

    # Find foo in files, recursively from current directory
    grep -r foo
    
    # From a specific directory
    grep -r foo /home/fred/Documents
    
    # Limit to specific file names
    grep -r --include="*.txt" foo
    
    # Search for 2 different words using ERE
    grep -E 'word1|word2' /path/to/file
    
    # Print file, removing blank lines and lines beginning with hash (#).
    grep -E -v '^#|^$' filename
    
    # Use grep in a test condition
    if grep -q 'pattern' file; then  #  The "-q" option causes nothing to echo to stdout.
      cmds;
    fi
    
    # List files that contain foo AND bar on different lines
    grep -rzl 'foo.*bar

## Options
    -V, --version             print version information and exit
    --help                    display help and exit
    -r, --recursive           search in current directory and all subdirectories below

### Pattern Syntax
    -E, --extended-regexp     Extended Regular Expressions (ERE)
    -F, --fixed-strings       pattern is a string, not a regex
    -P, --perl-regexp         Perl Compatible Regular expression (PCRE)

### Matching Control
    -e, --regexp=PATTERN      for specifying multiple patterns or if PATTERN begins with a '-'
    -i, --ignore-case         ignore case distinctions
    -v, --invert-match        select non-matching lines
    -w, --word-regexp         force PATTERN to match only whole words
    -z, --null-data           treat entire file as one line, substituting newlines with null character

### Output Control
    -c, --count               print only a count of matching lines per FILE
    --color=always            force color output when piped to another command.
    -l, --files-with-matches  print only names of FILEs that match.
    -m, --max-count=NUM       stop after NUM matches. often set to 1 for large files.
    -n, --line-number         print line number with output lines
    -o, --only-matching       show only the matching part of a line
    -q, --quiet, --silent     suppress all normal output

### Context
    -A, --after-context=NUM   print NUM lines of trailing context
    -B, --before-context=NUM  print NUM lines of leading context
    -C, --context=NUM         print NUM lines of output context

## Notes
egrep and fgrep are obsolete, Use grep -E or grep -F

To search compressed files, use **zgrep**, **zegrep**, or **zfgrep**.  
These also work on non-compressed files, though slower than grep  
Handy for searching through a mixed set of files, some compressed, some
not.

To search bzipped files, use **bzgrep**
