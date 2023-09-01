# Cut, Paste and Join

## Cut
Displays only specific columns from a text file or STDIN

### Examples

```
# Display the unix login names for all the users in the system.
cut -d ':' -f 1 /etc/passwd
# you can squash the arguments
cut -d: -f1 /etc/passwd

# Use space as delimiter, get 2nd and 4th fields
cut -d ' ' -f2,4
# another way to specify space as delimiter
cut -d\  -f2,4

# Use double quote as delimiter, get from field 3 to end of line
cut -d '"' -f3-
# or
cut -d\" -f3-

# Get first 4 fields
cut -d ' ' -f-4

# Get 1st field, and from 3rd field to end of line
cut -d ' ' -f1,3-

# Get first 20 characters of line. ie. truncate line at position 20
cut -c 20  # works best with ascii characters, not so well with unicode

### Cut vs Awk
- Cut supports only one delimiter character, TAB by default.
- If delimiters are repeated, cut still counts the null spaces in between as fields.
- Awk splits fields on whitespace by default and ignores repeated delimiters, which is
usually what you want.
- Awk can specify the delimiter with a regular expression.

## Paste
Like "cat" but operates "horizontally".

### Examples
    # no options, one file, acts like cat.
    paste file1
    Fred
    Barney
    Wilma
    Betty

    # Mash all lines into one line, with default tab delimiter
    paste -s file1
    Fred    Barney  Wilma   Betty

    # comma delimiter
    paste -s -d, file1
    Fred,Barney,Wilma,Betty

### Examples with two files

    cat file2
    Flintstone
    Rubble
    Flintstone
    Rubble

    # cat the two files "sideways" with default tab delimiter
    paste file1 file2  
    Fred    Flintstone
    Barney  Rubble
    Wilma   Flintstone
    Betty   Rubble

    # comma delimited
    paste -d, file2 file1
    Flintstone,Fred
    Rubble,Barney
    Flintstone,Wilma
    Rubble,Betty

## Join
The join command is similar to joining two tables in a database.

**Note:** Before joining the files, make sure to sort the
files on the joining fields.

### Example

    cat emp.txt
    10 mark
    10 steve
    20 scott
    30 chris
    
    cat dept.txt
    10 hr
    20 finance
    30 db

    join emp.txt dept.txt
    10 mark hr
    10 steve hr
    20 scott finance
    30 chris db

