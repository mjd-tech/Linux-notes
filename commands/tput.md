# tput and ANSI
Send ANSI escape sequences to the terminal.  
Useful in Bash scripts.

## Reading Terminal Attributes

| Command         | Description                       |
|-----------------|-----------------------------------|
| `tput longname` | Full name of the terminal type    |
| `tput lines`    | Number of lines in the terminal   |
| `tput cols`     | Number of columns in the terminal |
| `tput colors`   | Number of colors available        |

The lines and cols values are dynamic. That is, they are updated as the
size of the terminal window changes.

## Controlling The Cursor

| Command            | Description                                       |
|:-------------------|:--------------------------------------------------|
| `tput sc`          | Save the cursor position                          |
| `tput rc`          | Restore the cursor position                       |
| `tput home`        | Move the cursor to upper left corner (0,0)        |
| `tput cup row col` | Move the cursor to position row, col (zero based) |
| `tput cud1`        | Move the cursor down 1 line                       |
| `tput cuu1`        | Move the cursor up 1 line                         |
| `tput civis`       | Set to cursor to be invisible                     |
| `tput cnorm`       | Set the cursor to its normal state                |

Note: tput cursor position is zero-based, ie, the home position is 0,0.
ANSI escape sequences are one-based, home is 1,1

## Text Attributes

| Command              | Description                                 |
|:---------------------|:--------------------------------------------|
| `tput bold`          | Start bold text                             |
| `tput rev`           | Start reverse video                         |
| `tput blink`         | Start blinking text                         |
| `tput invis`         | Start invisible text                        |
| `tput smul`          | Start underlined text                       |
| `tput rmul`          | End underlined text                         |
| `tput smso`          | Start "standout" mode                       |
| `tput rmso`          | End "standout" mode                         |
| `tput sgr0`          | Turn off all attributes - Revert to default |
| `tput setaf <value>` | Set text foreground color                   |
| `tput setab <value>` | Set text background color                   |

Some capabilities only have a command to turn the attribute on. In these
cases, the sgr0 command can be used to return the text rendering to a
"normal" state.

## Colors

- Most consoles support 8 foreground text colors and 8 background colors.
- Most xterms support 256
- Use setaf and setab to set foreground and background colors

For most shell scripting it's best to stick to the standard 8

| Value | Color   |
|-------|---------|
| 0     | Black   |
| 1     | Red     |
| 2     | Green   |
| 3     | Yellow  |
| 4     | Blue    |
| 5     | Magenta |
| 6     | Cyan    |
| 7     | White   |

To see how many colors your terminal supports:

    tput colors

To see what the color numbers are on a 256 color terminal.

    for c in {0..255}; do tput setaf $c; echo color = $c; done

## Clearing The Screen

These commands allow us to selectively clear portions of the terminal
display:

| Command      | Description                                                        |
|--------------|--------------------------------------------------------------------|
| `tput smcup` | Save screen contents. (switch to your terminal's alternate screen) |
| `tput rmcup` | Restore screen contents. (switch back to terminal's main screen)   |
| `tput el`    | Clear from the cursor to the end of the line                       |
| `tput el1`   | Clear from the cursor to the beginning of the line                 |
| `tput ed`    | Clear from the cursor to the end of the screen                     |
| `tput clear` | Clear the entire screen and home the cursor                        |

Most Linux terminal emulation apps such as gnome-terminal, rxvt, xterm,
etc. support an "alternate" screen. Apps like less and vim will use the
alternate screen then switch back to the main screen when finished.
Thus, you don't see the results of your "less" or "vim" session.

## ANSI Codes

- All ANSI codes begin with ESC[ (escape,open bracket)
- ESC is 033 (octal) x1b (hex) 27 (dec)
- In Bash, escape is: `\e` or `\033` or `\x1b`

``` bash
# ANSI colors and styles:
# foreground (text) colors: Black, Red, Green, Yellow, Blue, Magenta, Cyan, White, Default
blk='\e[30m'    red='\e[31m'    grn='\e[32m'    yel='\e[33m'
blu='\e[34m'    mag='\e[35m'    cyn='\e[36m'    wht='\e[37m'  def='\e39m'

# light (aka bright) foreground colors:
lblk='\e[90m'   lred='\e[91m'   lgrn='\e[92m'   lyel='\e[93m'
lblu='\e[94m'   lmag='\e[95m'   lcyn='\e[96m'   lwht='\e[97m'

# background colors:
onblk='\e[40m'  onred='\e[41m'  ongrn='\e[42m'  onyel='\e[43m'
onblu='\e[44m'  onmag='\e[45m'  oncyn='\e[46m'  onwht='\e[47m'  ondef='\e49m'

# light background colors:
onlblk='\e[100m'  onlred='\e[101m'  onlgrn='\e[102m'  onlyel='\e[103m'
onlblu='\e[104m'  onlmag='\e[105m'  onlcyn='\e[106m'  onlwht='\e[107m'

# styles: Bold, Dim, Italic, Underline, Blink, Reverse, Hidden, Strike-through
bld='\e[1m'     dim='\e[2m'     itl='\e[3m'     unl='\e[4m'
bnk='\e[5m'     rev='\e[7m'     hid='\e[8m'     stk='\e[9m'
nobld='\e[22m'  nodim='\e[22m'  noitl='\e[23m'  nounl='\e[24m'
nobnk='\e[25m'  norev='\e[27m'  nostk='\e[29m'

# reset all colors and styles to default
rst='\e[0m'

# codes can be combined
# Example: black fg on cyan background
'\e[30;46m'
```

Usage:

    echo -e "${bld}${red}This text in bold red. ${def}Just bold. ${nobld}Normal text."

### In practice
Most of the time the following is adequate:  
red (error,failure), green (success), yellow (notice), bold and reset.

```bash
red='\e[31m' grn='\e[32m' yel='\e[33m' bld='\e[1m' rst='\e[0m'

# a "press any key to continue" function, bold yellow, that auto clears itself

Pause () {
    echo -e "${bld}${yel}Press any key to continue...${rst}"; read -rsn 1;
    # Move cursor up 1 line, clear to end of line, clear to beginning of line; put new line
    tput cuu1; tput el; tput el1; echo
}

# display notice
echo -e "${yel}Syncing files. This may take a while...${rst}"

# display success/failure
if rsync -av /source/dir/ /dest/dir; then
    echo -en "${grn}${bld}Sync successful.${rst}"
else
    echo -e "${red}${bld}ERROR: Sync failed.${rst}"
fi

# die function. Usage: Die "message here"
Die() {
    echo "${bld}${red}${1:-"Error"}${rst}" >&2
    exit 1
}
```

## Further Reading
[Bash Hacker's Wiki](http://wiki.bash-hackers.org/scripting/terminalcodes/)

[Greg's Wiki](http://mywiki.wooledge.org/BashFAQ/037)

[Bash Prompt HOWTO](http://www.tldp.org/HOWTO/Bash-Prompt-HOWTO/x405.html)
