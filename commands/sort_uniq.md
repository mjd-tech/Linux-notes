# sort and uniq

## sort
Write sorted concatenation of all FILE(s) to standard output.

By default, sorts on the first character of line. If there are
duplicates, it looks at the second character, and so on...

### Examples

```
sort file.txt
somecmd | sort
sort -r file.txt        # reverse order
LC_ALL=C sort file.txt  # Ascii order (capitals before small letters)
sort -f file.txt        # ignore case
sort -u file.txt        # unique (don't output duplicate lines)
sort -n numbers.txt     # Numeric sort
sort -h numbers.txt     # "Human-readable" numbers (1K, 5M, 12G, etc)
sort -n -c numbers.txt  # check if already sorted
sort -k2 file.txt       # sort on the 2nd column (space delimited)
sort -t: -k3 /etc/passwd    # sort passwd file on user id (3rd field, : delimited)
```
## uniq
Output the unique lines from stdin or file.

'uniq' does not detect repeated lines unless they are adjacent.  
It is common to use `sort | uniq` or `sort -u`

### Examples

```
# Remove duplicate lines
sort path/to/file | uniq

# Display only unique lines:
sort path/to/file | uniq -u

# Display only duplicate lines:
sort path/to/file | uniq -d

# Display number of occurrences of each line along with that line:
sort path/to/file | uniq -c

# Display number of occurrences of each line, sorted by the most frequent:
sort path/to/file | uniq -c | sort -nr
```
