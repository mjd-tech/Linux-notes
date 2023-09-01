# wc
Prints count of words, characters, or lines in files or from standard input.

A **word** is a non-zero-length sequence of characters delimited by white space.


## Options

if no options specified, wc prints: newlines, words, characters

| Option                | Command                                                                                                              |
|-----------------------|----------------------------------------------------------------------------------------------------------------------|
| -l, --lines           | print newline counts                                                                                                 |
| -w, --words           | print the word counts                                                                                                |
| -c, --bytes           | print byte counts                                                                                                    |
| -m, --chars           | print character counts                                                                                               |
| -L, --max-line-length | print the length of the longest line                                                                                 |
                                                                  |

Beware of Unicode characters. For example: ñ is one character, but 2 bytes
    echo -n piñata | wc -m
    # Result: 6
    
    echo -n piñata | wc -c
    # Result: 7
    


