# Filter Rules
Filter rules tell rsync which files and directories to include and exclude. 

Most common are exclude filter rules, commonly called "excludes"

Typically, you exclude:
- cache directories
- temp directories
- mount points
- transient system directories (/dev /proc /run /sys)
- any stuff you don't need to be copied.

There are also "include" filter rules.  
A common use is to list _include_ rules first,  
then as the last rule, exclude `*` (everything).
Include rules can be tricky to use (see below)

## Specify excludes on command line
You can use one or more --exclude= statements.
```bash
rsync -avx --exclude=foo.txt --exclude=bar.txt SRC DEST
```
Or use one --exclude= statement like this
```bash
rsync -avx --exclude={foo.txt,bar.txt} SRC DEST
```

## Excludes file
You can put filter rules in a file.  
In the rsync command, put `--exclude-from=filename` or `--include-from=filename`  
`filename` can contain both include and exclude rules.  
So it is common to just put all rules in an "excludes" file and call it with
`--exclude-from=filename`

```bash
rsync -avx --exclude-from=my_excludes.txt SRC DEST
```

## Excludes files format
- one filter rule per line
- blank lines ignored
- lines beginning with `#` are comments, and ignored
- lines beginning with `+` are include rules, be careful (see below)
- lines beginning with `-` are exclude rules. 

In an excludes file, the leading `-` is implied, so you can omit it.

## Patterns
- Filter Rules can use patterns similar to shell wildcards
- All patterns are **relative** to the source path given on the command line.

Rsync chooses between doing a simple string match and wildcard
matching by checking if the pattern contains one of these three
wildcard characters: `*` `?` `[`

| Wildcard | Description                                                    |
| :------- | :------------------------------------------------------------- |
| `*`      | matches any path component, except slash /                     |
| `**`     | matches any path component, including slashes                  |
| `?`      | matches any single character, except slash /                   |
| `[`      | introduces a character class, such as `[a-z]` or `[[:alpha:]]` |
| `\`      | escape a wildcard character.                                   |

Examples:
| Pattern      | Description                                     |
| :----------- | :---------------------------------------------- |
| `foo`        | file or directory named **exactly** foo         |
| `foo*`       | **begins with** foo: foo, foobar, foobar.baz    |
| `*foo`       | **ends with** foo: foo, barfoo, bar.foo         |
| `*foo*`      | **contains** foo: foo, food, barfoo.baz         |
| `foo?`       | food, fool foot                                 |
| `foo[dl]`    | food, fool, but not foot                        |
| `/foo`       | foo, "anchored" at the "root of the transfer"   |
| `foo/`       | directory named exactly foo, but not a file     |
| `foo/*`      | include foo/ but exclude everything within foo/ |
| `foo/*/bar`  | bar, exactly one subdirectory under foo/        |
| `foo/**bar`  | bar, zero or more subdirectories under foo/     |
| `foo/**/bar` | bar, one or more subdirectories under foo/      |

## Include rules
Example: 
- you want to rsync your Documents directory
- exclude everything in `Documents/stuff` except `special_file.pdf`

The rsync command
```bash
rsync -av --exclude-from=excludes.txt $HOME/Documents/ remote_host:/Documents
```
excludes.txt file, first attempt
```
+ /stuff/special_file.pdf
/stuff/*
```
This **doesn't work** as expected. 
It copies a blank `stuff` directory, and does **not** copy special_file.pdf

You have to first include the `stuff` directory.

excludes.txt file, correct
```
+ /stuff/
+ /stuff/special_file.pdf
- /stuff/*
```
You have to include the **parent** directory first, then include the file you want, then exclude everything else.