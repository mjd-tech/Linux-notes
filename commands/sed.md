# sed
Non-interactive **s**tream **ed**itor. Reads standard input or file(s)

## Command-Line Options

    sed OPTIONS... [SCRIPT] [INPUTFILE...]

SCRIPT is usually enclosed in single quotes to protect from shell expansion.

```
-n --quiet --silent          Suppress the default output. Use p command to print specific lines
-e 'cmd' --expression='cmd'  Next argument is an editing command. Used when multiple commands are specified.
-E -r --regex-extended       Use extended regular expressions, like egrep.
-i[SUFFIX] --in-place[=SUFFIX]  overwrite original file, optionally making a backup named filename.SUFFIX
```

## Substitute (find and replace)

Each example replaces "foo" with "bar"
```
sed 's/foo/bar/'                    # replace only 1st instance in a line.
sed 's_foo_bar_'                    # same, use a different delimiter than /
sed 's/foo/bar/4'                   # replace only 4th instance in a line.
sed 's/foo/bar/g'                   # replace ALL instances in a line.
sed 's/foo/bar/4g'                  # replace 4th to last instance in a line.
sed '10 s/foo/bar/'                 # replace only on line 10.
sed '5! s/foo/bar/'                 # replace on all lines except line 5
sed '1,10 s/foo/bar/'               # replace on lines 1-10.
sed '11,$ s/foo/bar/'               # replace on lines 11 to end of file.
sed '/baz/ s/foo/bar/'              # replace only on lines with "baz".
sed 's/\(.*\)foo\(.*foo\)/\1bar\2/' # replace the next-to-last instance.
sed -E 's/(.*)foo(.*foo)/\1bar\2/'  # same, easier to read
sed -E 's/(.*)foo/\1bar/'           # replace only the last instance.
sed '/start/,/stop/ s/foo/bar/'     # replace on lines from start to stop, inclusive.
sed '/start/,/stop/ {//! s/foo/bar/;}'   # replace BETWEEN start and stop, exclusive:
```
Note:  
- Brackets {} are used to group commands triggered by a single address (or address-range) match. 
- The empty regular expression '//' repeats the last regular expression match.
- Parentheses () are used to "capture" a portion of the regex, so it can be "backreferenced" in the replacement string.

## Misc sed snippets

```
# Remove blank lines
sed '/^$/d'
sed '/^$/ d'               # same, easier to read

# Remove whitespace (spaces, tabs)
sed 's/^\s*//'              # Remove whitespace from beginning of each line.
sed 's/\s*$//'              # Remove whitespace from end of each line.
sed 's/^\s*//; s/\s*$//'    # Remove whitespace from beginning and end.

# Remove lines beginning with comments, blank lines, and lines with only whitespace
sed '/\s*#/d; /^\s*$/d'             # two sed commands separated with ;

sed -n '10p;10q'                    # Print only 10th line.
sed -n '10,20p;20q'                 # Print lines 10 to 20.
sed -E 's/(.*)A/\1B/g'              # Modify anystringA to anystringB.
sed ':a; /\\$/ N; s/\\\n//; ta'     # Concatenate lines with trailing \.
sed 's/.$//' filename               # convert DOS line endings to Unix.
sed G filename                      # double space a file (add blank line after all lines)

sed '/foo/ i text'                  # insert text before lines matching foo
sed '/foo/ a text'                  # append text after  lines matching foo
sed '/foo/ c text'                  # change lines matching foo to text
sed '/foo/ c\ text'                 # text with leading space
sed '/foo/ c\    text'              # 4 leading spaces, only the first needs to be escaped

# Work with files
sed 's/foo/bar/' file               # read from file, send to stdout.
sed 's/foo/bar/' file > newfile     # read from file, write to newfile.
sed -i 's/foo/bar/' file            # edit in place. GNU sed only.
sed -i.bak 's/foo/bar/' file        # as above, but first create file.bak
```

## Greedy matching
`.*` gives the longest possible match (greedy). This can be a problem, especially
with groups/backreferences.

Example: remove bold tags.
```
# Greedy matching, removes tags but also the word in between
echo "<b>foo</b>bar" | sed 's/<.*>//g'
bar

# Non greedy matching, using "negated character class"
echo "<b>foo</b>bar" | sed 's/<[^>]*>//g'
foobar
```

Another option: use _perl_ instead of _sed_. Perl has a "non-greedy match", `.*?`

```
echo "<b>foo</b>bar" | perl -pe 's/<.*?>//g'
foobar
```

Note: The "negated character class" technique only works with a **single character**  
Sometimes you need a " multi-character negated character class".  
Example: remove the first bolded word (and its tags) on a line.

```
echo "<b>foo</b>bar<b>baz</b>" | sed 's|</b>|\a|; s|<b>[^\a]*\a||'
bar<b>baz</b>
```
Explanation:  
Use `|` instead of `/` for the substitute command.  
substitute the closing bold tag `</b>` with a single character.
Make sure this character is not used anywhere else. A good choice is the "alert" ascii character, \a.
This is almost never used.
Then, use the special character with the "negated character class" method.

## Efficiency

    # This works but can be slow
    sed 's/old/new' hugefile
    
    # More efficient
    sed '/old/ s//new/g' hugefile 

In the "s" command, the empty regular expression ‘//’ repeats
the last regular expression match. In this case "old"

Firing up the substitute sequence is relatively "expensive".
This method only runs it on lines that need it.

### Using conditional branching

Add commas to all numeric strings in a file, changing "1234567" to
"1,234,567"
```
echo 123456789 | sed -r ':a; s/\B[0-9]{3}\b/,&/; ta'
# Result: 123,456,789

# Explanation
:a                    set a label

s/\B[0-9]{3}\b/       find "non word boundary", 3 digits, word boundary.
                      ie. find the rightmost 3 digits provided no leading comma or blank.

,&/                   replace it with a leading comma then the regexp match

ta                    If a substitution made, goto label, otherwise quit.

Note:
A "word" character is a-Z 0-9 or _ (underscore).
A "word boundary" is the null space between a "word" character and a "non-word" character.
A "non-word boundary" has "word" characters both left and right, or "non-word" characters both left and right.
```
