# less - terminal pager
Display files or standard input in a pager, with search capability

## Examples
    less -M readme.txt                     # Read "readme.txt.", with "long prompt"
    less +F /var/log/mail.log              # Follow mode for log
    file * | less                          # Easier file analysis.
    
## Syntax
    less [options] [file_name]

## Frequently used options

    -N: Shows line numbers (useful for source code viewing).
    -R: Display ANSI colors
    -S: Disables line wrap. Long lines can be seen by side scrolling.
    -X: Don't clear the screen upon exit.
    -i: Use case insentive search
    --help: Shows help. In less itself!

## Navigation

| Key                        | Command                                                                |
|----------------------------|------------------------------------------------------------------------|
| Space bar or Page Down     | Next Page                                                              |
| b or Page Up               | Previous Page                                                          |
| v                          | Edit current file in default editor. Does not work with stdin          |
| j or ↵ Enter or Down Arrow | Next Line                                                              |
| k or Up Arrow              | Previous Line                                                          |
| g                          | Go to First Line                                                       |
| G                          | Go to Last Line                                                        |
| 10G                        | Go to Line 10                                                          |
| /regex                     | Forward Search for regex                                               |
| ?regex                     | Backward Search for regex                                              |
| &regex                     | Show only matching lines. Use & alone to return to normal.             |
| n                          | Next Search Match                                                      |
| N                          | Previous Search Match                                                  |
| Esc u                      | Turn off Search Highlighting                                            |
| -i                         | Toggle "smart case" (case insentive unless pattern contains uppercase) |
| -M                         | Toggle long prompt. More detail about current file position            |
| = or Ctrl+G                | File information                                                       |
| h                          | Help. This is presented with less, q to quit.                          |
| q                          | Quit                                                                   |

