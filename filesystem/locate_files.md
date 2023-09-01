# Locating files

locate, which, whereis, apropos

## locate
```
locate foo          files and directories that contain *foo*. Equal to *foo*
locate -i foo       case-insensitive, matches *foo Foo fOO*, etc.
locate -b foo       basename. Match only against the file name.
locate -r '/foo\$'  use regular expression to get an exact match for *foo*
```
locate relies on a database to function. By default, the db is updated daily via cron.
To manually update: `sudo updatedb`

## which
find the location or existence of an executable in the current PATH
```
which firefox
# result: /usr/bin/firefox or NOTHING if firefox not installed
```

## whereis
similar to *which* but also locates the source, man page, and other associated directories.

```
whereis firefox
# result: /usr/bin/firefox /usr/lib/firefox /usr/lib64/firefox /usr/share/firefox
```

## apropos
Use when you can't remember the name of a command that does a certain function.
*apropos* searches the manual pages description field.
For example, you forgot the name of the media player:

    apropos player

## fd
Note: fd is installed as fdfind on Debian based systems, Ubuntu, Linux Mint,
to prevent a conflict with the "fdclone" package, which is not installed by default and
you probably don't use.

Fix it with: `ln -s $(which fdfind) ~/.local/bin/fd`

fd does most of what find does, with a more "sane" syntax, also
- hidden files and directories are skipped by default (unlike find)
- if you're searching within a git repository, fd respects .gitignore
- you can't just put a .gitignore file in any old folder
- you CAN put a .ignore or .fdignore file, same syntax as a .gitignore
- can put a global ignore file at $HOME/.config/fd/ignore
- fd does not do "or" operations like `find -o`
- fd does not search for permissions like `find -perm`

```
- Recursively find files matching a specific pattern in the current directory:
  fd "string|regex"

- Find files that begin with `foo`:
  fd "^foo"

- Find files with a specific extension:
  fd --extension txt

- Find files in a specific directory:
  fd "string|regex" path/to/directory

- Include ignored and hidden files in the search:
  fd --hidden --no-ignore "string|regex"

- Execute a command on each search result returned:
  fd "string|regex" --exec command
```

## grep - find text within files

grep searches the input files for lines that match a given pattern. When
it finds a match, it copies the line to standard output.

| Command                    | Description                                              |
|----------------------------|----------------------------------------------------------|
| `grep foo somefile`        | show all lines containing the text "foo" in somefile     |
| `grep foo /home/fred/*`    | as above, but in all files in Fred's home directory      |
| `grep -r foo /home/fred/*` | (recursive) as above, but also search all subdirectories |
| `grep -v foo somefile`     | show all lines NOT containing "foo"                      |
| `grep -c foo somefile`     | count the number of lines containing "foo"               |
| `grep -n foo somefile`     | display the line number of lines containing "foo"        |
| `grep '\bfoo\b' somefile`  | find the word "foo", not words like "food" or "barfoo"   |

- grep uses regular expressions in the search pattern.
- enclose regex in single quotes, prevents the shell interpreting the special characters.
- if searching for a simple string in a large number of files, use `grep -F ...`
This disables the regex engine and is faster.

