# tr
Copy standard input to standard output substituting or deleting selected characters.  
tr cannot open files, it works only with stdin.

## Examples
**Translate braces into parentheses**
    tr '{}' '()' < textfile > newfile

**Replace successive blanks (space & tab), with a single space**  
This is useful as a pre-processor for cut.

    some_command | tr -s '[:blank:]' ' ' | cut -d ' ' -f2

**Translate lowercase characters to uppercase**

    tr 'a-z' 'A-Z' < textfile > newfile
    OR
    tr '[:lower:]' '[:upper]' < textfile > newfile

**Remove carriage returns from Dos/Windows text files**

    tr -d '\015' < pc.file > unix.file
    OR
    tr -d '\r' < pc.file > unix.file

**Remove all non-printable character from a file**

    tr -cd [:print:] < textfile

**Replace successive newlines with a single new line**

    tr -s '\n' < textfile > newfile
    OR
    tr -s '\012' < textfile > newfile

**Join all the lines in a file into a single line**

    tr -s '\n' ' ' < textfile > newfile

**Sanitize input so a user cannot submit commands as arguments in a
script**

    tr --delete '=;:`"<>,./?!@#$%^&(){}[]'

**Create a sorted list of the unique words contained in a text file, one
word per line**

    cat textfile | tr -cs "[:alnum:]" "\n" | sort | uniq -c | sort -rn

## Syntax

      tr [OPTIONS] SET1 [SET2]

If both the SET1 and SET2 are specified and '-d' OPTION is not
specified, then tr command will replace each character in SET1 with
the character in same position in SET2. If SET1 is longer than SET2,
then SET2 is extended by repeating its last character as necessary. If
SET2 is longer, then excess characters in SET2 are ignored.

| Option                | Description                                                                                                                       |
|-----------------------|-------------------------------------------------------------------------------|
| -c, -C, --complement  | operations apply to characters *not* in the given set                         |
| -d, --delete          | delete characters in set1, do not translate                                   |
| -s, --squeeze-repeats | squeeze repeated characters in the output into a singe character.             |
| -t, --truncate-set1   | Truncate set1 to the length of set2. This option reverse the default behavior |

## tr SET Notation

Sets are specified as strings of characters. Most represent themselves.
Interpreted sequences are:

    \nnn       -- character with octal value nnn
    \xnn       -- character with hexadecimal value nn
    \\         -- backslash
    \a         -- alert
    \b         -- backpace
    \f         -- form feed
    \r         -- return
    \t         -- horizontal tab
    \v         -- vertical tab
    \E         -- escape
    c1-c2      -- all characters from c1 to c2 in ascending order. The character specified by c1 must collate before the character specified by c2.
    [c1-c2]    -- same as c1-c2 if both sets use this form
    [c*]       -- set2 extended to the length of set1 with the symbol c. In other words fills out the set2 with the character specified by c. 
                  This option can be used only at the end of set2. Any characters specified after the * (asterisk) are ignored.
    [c*N]      -- N copies of symbol c. N is considered a decimal integer unless the first digit is a 0; then it is considered an octal integer.
    [:alnum:]  -- all letters and digits
    [:alpha:]  -- all letters
    [:blank:]  -- all horizontal whitespace
    [:cntrl:]  -- all control characters
    [:digit:]  -- all digits
    [:graph:]  -- all printable characters, not including space
    [:lower:]  -- all lower case letters
    [:print:]  -- all printable characters, including space
    [:punct:]  -- all punctuation characters
    [:space:]  -- all horizontal or vertical whitespace
    [:upper:]  -- all upper case letters
    [:xdigit:] -- all hexadecimal digits
    [=c=]      -- Specifies all of the characters with the same equivalence class as the character specified by C.

Notes:

    Only [:lower:] and [:upper:] are guaranteed to expand in ascending order. They can be used in pairs to specify case conversion.
    -s (Squeeze all strings of repeated output characters to single characters) uses set1 if neither translating nor deleting specified,
        otherwise squeeze uses set2 and occurs after translation or deletion.

## Difference between tr and sed
Don't use tr to do word substitution.
    # Don't do this
    tr 'foo' 'bar'
    
    # Do this
    sed 's/foo/bar/g'

tr performs character based transformation but sed performs string based
transformation.

