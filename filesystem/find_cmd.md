# find
Recursively Locate files that meet certain criteria, such as name, size, age, and  type.
- Does NOT look at the contents of a file, use `grep` or `ripgrep` for that.
- finds hidden files and directories by default. Clumsy syntax to exclude these.
- `fd` is an easier to use alternative

## General Syntax

    find /path <search criteria> <action>

- path can be: full path; relative path; nothing; a dot . (search from current directory)
- If path is blank, search from current directory
- If file matches "search criteria", perform "action" on each found file.

## Search Criteria

Here are some of the most useful search criteria options:

| Option                 | Explanation                                                                          |
|----------------------  |--------------------------------------------------------------------------------------|
| `-name 'pattern'`      | Uses "shell" patterns. Cannot include slashes / in the pattern                       |
| `\! -name 'pattern'`   | Files that DON'T match 'pattern' (must escape the !)                                 |
| `-not -name 'pattern'` | Same as above. Can use either `\!` or `-not` to negate a search criteria             |
| `-iname 'pattern'`     | Same, case-insensitive.                                                              |
| `-regex 'pattern'`     | Use regex instead of shell pattern                                                   |
| `-type filetype`       | Files of a specific type, normally -type f (regular files) or -type d (directories)  |
| `-size +4G`            | larger than 4 Gibibytes  (1024 based)                                                |
| `-size -1k`            | smaller than 1 Kibibytes (1024 based)                                                |
| `-mtime 0`             | modified within past 24 hours                                                        |
| `-mtime -7`            | modified within past 7 days                                                          |
| `-mtime +30`           | modified 30 or more days ago                                                         |
| `-mmin -10`            | modified within past 10 minutes                                                      |
| `-user uname`          | owned by user. uname can be username (fred) or user id number (1000)                 |
| `-group gname`         | owned by group. gname can be groupname (fred) or group id number (1000)              |
| `-perm mode`           | EXACTLY match mode. eg. -perm 664                                                    |
| `-perm -mode`          | match AT LEAST mode. eg. -perm -664 also matches 777                                 |
| `-perm /mode`          | match BITS in mode. eg. -perm /220 writable by either owner or group                 |


File Size suffix:
- b 512-byte blocks (this is the default if no suffix is used)
- c bytes
- w two-byte words (rarely used)
- k kibibytes (KiB, units of 1024 bytes)
- M mebibytes (MiB, units of 1024 * 1024 = 1048576 bytes)
- G gibibytes (GiB, units of 1024 * 1024 * 1024 = 1073741824 bytes)

## Actions

| Option               | Explanation                                                                          |
|----------------------|--------------------------------------------------------------------------------------|
| `-delete`            | deletes files and empty directories                       |
| `-exec somecmd {} +` | executes command on each item found |
| `-print`             | This is the default action, but sometimes you need to specify it. |
| `-print0`            | Use this when piping into `xargs -0` to handle spaces in filenames. |
| `-printf 'fmt'`      | print things other than full filename, see man page for options |
| `-prune dir`         | don't process dir. Tricky to use. See below. |

### Basic Examples

In Linux, everything is a "file", so it is best to use a modifier such as `-type f` to be specific.

Note: Wildcard characters `* ?` Parentheses `()` Bang `!` and semi-colon `;` MUST be escaped from the shell.

|Command                                                    |Explanation                                                                            |
| ---                                                       | ---                                                                                   |
| `find`                                                    | All files in current directory and all subdirectories                                 |
| `find .`                                                  | Same - preferred syntax                                                               |
| `find /somedir`                                           | All "files" in the somedir directory and all subdirectories                           |
| `find /`                                                  | All "files" on this computer. Not recommended. Slow.                                  |
| `find . -name 'foo'`                                      | A "file" named exactly _foo_                                                          |
| `find /opt /usr -name 'foo'`                              | search multiple directories                                                           |
| `find . -iname 'foo'`                                     | Same, case-insensitive                                                                |
| `find . -name 'foo.*'`                                    | files named _foo_, with any extension.                                                |
| `find . -name '*.txt'`                                    | All files with _.txt_ extension                                                       |
| `find . -type f -name '.*'`                               | Hidden files                                                                          |
| `find . -type d -name '.*'`                               | Hidden directories                                                                    |
| `find . -type f \! -name '*.txt'`                         | All files **not** ending in _.txt_                                                    |
| `find . -type f -not -name '*.txt'`                       | Same, but not POSIX compliant syntax                                                  |
| `find . -type f \( -iname '*.jpg" -o -iname '*.jpeg' \)`  | find all `jp[e]g` files, case insensitive.                                            |
| `find . -type f \( -iname '*.jpg' -or -iname '*.jpeg' \)` | Same but `-or` is not POSIX compliant syntax                                          |
| `find . -type f -exec chmod 644 {} \;`                    | The final semi-colon is part of the "find" command and must be escaped from the shell |
| `find . -type d -exec chmod 775 {} \;`                    | Change directory permissions to default                                               |
| `find . -type f -size +10000 -exec ls -al {} \;`          | Long list files larger than 10,000 bytes                                              |
| `find . -empty`                                           | Empty files (zero bytes), mostly lock-files and place holders                         |
| `find . -newer reference_file`                            | files modified after reference_file was last modified                                 |
| `find . ! -readable -prune -o -type d -print`             | prevent "Permission denied" errors ie. lost+found                                     |

### Limit Search To Specific Directory Level

Use the *mindepth* and *maxdepth* criteria. A depth of "1" means "the
directory given on the command line". mindepth and maxdepth must be before other search
criteria such as -name and -type

| Command                                         | Explanation                                |
|-------------------------------------------------|--------------------------------------------|
| `find /usr -maxdepth 1 -name "foo"`             | Do not search subdirectories               |
| `find /usr -mindepth 2 -name "foo"`             | Search subdirectories, but not /usr itself |
| `find /usr -mindepth 2 -maxdepth 3 -name "foo"` | Search subdirectories, but not below       |

### Exclude Directories
Note: `fd` is a better choice for this. find's syntax is unintuitive and easy to get wrong!

Find files in current directory and below, exclude .git directory.  
Method 1: "prune"
```
find . -type f -print -o -path ./.git -prune
# Can specify path first, but this is less intuitive
find . -path ./.git -prune -o -type f -print
```
- you must use `-print` or else find implicitly prints everything.
- prune is an "action", not a search criteria
- here, prune means "don't look in this directory"

Method 2: "not path"
```
find . -type f ! -path './.git/*'
find . -type f -not -path './.git/*'
```

- It's not enough to say ` ! -path ./.git` it needs the trailing `/*`
- "not path" is a search criteria, not an "action"
- less efficient that prune, but maybe slightly more intuitive.
- here, find does look inside the .git directory, but doesn't print its contents.

Example: find all JPEG files in home dir that are NOT in the Pictures directory.

```
cd  # get to the top level of your home directory
find . -iname '*.jpeg' -print -o -iname '*.jpg' -print -o -path ./Pictures -prune
# you can group the search criteria and use only one -print statement
find . \( -iname '*.jpeg' -o -iname '*.jpg' \) -print -o -path ./Pictures -prune
```

- you must escape `(` and `)` with a backslash
- these characters have special meaning to the shell

To exclude multiple directories, you can add extra `-o -path ... -prune` statements  
Example: find all txt files in home dir that are NOT in Downloads or Documents

```
cd
find . -iname '*.txt' -print -o -path ./Downloads -prune -o -path ./Documents -prune
# you can group the path statements and use only one -prune
find . -iname '*.txt' -print -o \( -path ./Downloads -o -path ./Documents \) -prune
```

### Combining find With xargs

- more efficient than using `-exec`option, if many files are involved.
- use the `-print0` option in find, and the `-0` option in xargs
- this correctly handles filenames with spaces.

```
find /somedir -name "*.txt" -print0 | xargs -0 ls -l
```


GNU find supports an alternate syntax that operates similar to piping
into xargs:

    find /somedir -name "*.txt" -exec ls -l {} +

- same as the regular `-exec` syntax, only the terminator is a `+` instead of `;`
- `-exec` should be the last argument to find.
- anything following `-exec` is the command to run and its arguments
- {} is a placeholder for "each file". It correctly handles files with spaces.
- the `+` does not need to be escaped from the shell
- This works well for simple finds
- but not so good if you have `-o` (or) statements in your find command.

