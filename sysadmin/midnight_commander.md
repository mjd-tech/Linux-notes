# Midnight Commander (mc)

> :warning: Make sure your terminal emulator does **NOT** map:  
> function keys **F1 - F10** (mc depends upon these keys)  
> **Ctrl-PgUp** (go up a directory)  

> :bulb: Tip: Turn off command prompt in Options...Layout.  
> Puts mc in "quick search" mode. Type beginning of file/directory...

| Keybinding      | Action                                                              |
|:----------------|:--------------------------------------------------------------------|
| Ctrl + o        | toggle command prompt/panels                                        |
| Alt + i         | show the **current directory** in the other panel                   |
| Alt + o         | open **currently selected directory** in **other panel**,           |
| Ctrl + PgUp     | move to the parent directory                                        |
| Ctrl + space    | show the size of currently selected directory (usually shows 4096)  |
| Ctrl + l (ell)  | refresh display                                                     |
| Alt + ,         | toggle vertical/horizontal split                                    |
| Alt + t         | switch the panel’s listing mode: default, brief, long, user-defined |
| Ctrl + u        | swap panels                                                         |
| Alt + s         | (when cmd prompt enabled) quick search mode. Accepts wildcards `* ?` |

## Command line stuff

mc uses Tab to switch panels.  
When you use the command line, and want to use Tab (for autocompletion)  
hit Esc, then Tab. Make sure you set Options...Configuration...**Complete - show all**

| Keybinding                  | Action                                               |
|:----------------------------|:-----------------------------------------------------|
| Esc + Tab                   | Autocomplete command line                            |
| Ctrl + Enter or Alt + Enter | copy the currently selected file’s name to the shell |
| Ctrl + Shift + Enter        | same as above, but the full path is copied           |
| Ctrl + X P                  | copy path of active panel to command line            |
| Ctrl + X Ctrl + P           | copy path of inactive panel to command line          |

## Options

**Configuration**  
Verbose operation and Compute totals - so that operations like copy/move
have a more detailed progress dialogs.

**Layout**  
Uncheck Hintbar visible - hints get annoying after a while.  
Uncheck command prompt. Now, you are in "quick search" mode.
use Alt-c (quick cd) to change dirs. 

**Panel options**  
Show backup files and Show hidden files - I keep both enabled, as I
often work with configuration files, etc.  
Lynx-like motion - left arrow goes to parent directory, right arrow
enters the directory under selection.  
File highlight - File types is useful, as it uses a different color for
example for executable files.

**File Size in K,M,G**  
The directory listing can be displayed in several different formats.
Press Alt-t to cycle through them. You can modify the "Custom" view:

The file to edit is ~/.config/mc/panels.ini.

To list the file sizes as K, M or G use a narrow size column using the
user_format key:

    [New Left Panel]
    user_format=half type name mark size:4 space mtime

    [New Right Panel]
    user_format=half type name mark size:4 space mtime

**Useful tip**  
set up a different skin when logged in as the root user. It’ll be easier
to differentiate between root’s and normal user’s session, when you’re
swapping between them (as is often the case).

## SMB support

Don't bother. You have to compile from source.
This only supports SMB1, has been basically abandoned by the developers,
and it never worked right in the first place.

## McEdit

To select an area for copy/move/delete:

- Put cursor at beginning of selection
- Hit F3. This toggles selection mode.
- use left, right arrow keys to select words on a line
- use up, down arrow keys to select lines.
- Hit F3 again to end selection

At this point F8 will delete the selection.

To copy or move the selection, put the cursor at the destination, and
hit F5 or F6.

The original text remains selected. This can be a distraction. To clear
the selection, hit F3 twice.  
This begins a new selection, selects nothing, then ends selection,
resulting in nothing selected.

Use Shift-F3 for "block selection". Use arrow keys to select the block.
End selection with either F3 or Shift F3

## Merging Files

It can compare folders and files (check the Command menu, F9 via
keyboard)

When comparing files, use **Enter** to navigate to the next diff block,
**F5** to merge the selected diff from the **right** to the **left**.

## The User Menu

When we press the F2 key, by default, Midnight Commander looks for a "menu" file in 

- Local menu - curent directory: `.mc.menu` (rarely used)
- User menu: `~/.config/mc/menu` 
- System directory: `/etc/mc/mc.menu`

The default User menu contains too much stuff that you'll never use.  

### Editing the User Menu

- Within mc, goto top menu...Command...Edit Menu File
- You get a choice, "Local" and "User"
- Choose User. This edits `~/.config/mc/menu`

> You can also edit the user menu directly, using your text editor of choice.  
> Make sure mc is NOT RUNNING while you do this.


### Menu File Format

A menu file consists of one or more entries. Each entry contains:

- A single character (usually a letter) that will act as a **hot key**
  for the entry when the menu is displayed.
- Following the hot key, on the same line, is the menu entry as it will
  appear on the menu.
- On the following lines, one or more commands to be performed when the
  menu entry is selected.
- These are ordinary shell commands. Any number of commands may be
  specified, so quite sophisticated operations are possible.
- Each command must be **indented** by at least one space or tab.
- A blank line separates menu entries.
- Comments may appear on their own lines. Each comment line starts with
  a # character.

Here is an example user menu entry that creates an HTML template in the
current directory:

    # Create a new HTML file

    H   Create a new HTML file
        { echo "<html>"
        echo "\t<head>\n\t</head>"
        echo "\t<body>\n\t</body>"
        echo "</html>"; }  > new_page.html

Midnight Commander uses **sh** when it executes user menu commands,
**not Bash**. So make sure you don't use any "Bashisms"

### Macros

When Midnight Commander encounters one of these macros, it substitutes
the value the macro represents. Here are the most commonly used macros:

List of common macros

| Macro | Meaning                                                             |
|:------|:--------------------------------------------------------------------|
| %f    | current file in the **selected** panel                              |
| %F    | current file in the **unselected** panel                            |
| %x    | extension of current file name                                      |
| %b    | current file name without extension                                 |
| %d    | current directory name                                              |
| %D    | Same, for unselected panel                                          |
| %t    | currently tagged files                                              |
| %T    | Same, for unselected panel                                          |
| %s    | If files are tagged, they are used, else the selected file is used. |
| %S    | Same, for unselected panel                                          |

**Capital** letters refer to the **unselected** panel

**Example:**  
create a user menu entry that would resize a JPEG image using the
*convert* program from the ImageMagick suite. Using macros, we could
write a menu entry like this, which would act on the currently selected
file:

    #   Resize an image using convert

    R   Resize image to fit within 800 pixel bounding square
        size=800
        convert "%f" -resize ${size}x${size} "%b-${size}.%x"

Using the %b and %x macros, we are able to construct a *new output file
name* for the resized image.  
There is still one potential problem with this menu entry.  
It’s possible to run the menu entry command on a directory, or a
non-image file (Doing so would not be good).

Midnight Commander provides a method for only displaying menu entries
appropriate to the currently selected (or tagged) file(s).

### Conditions

Conditions determine whether to:

- Display a menu item or not. (addition condition)
- Make a menu item the default choice. (default addition)

A condition is added to a menu entry just before the first line. A
condition starts with either:

- \+ (for an addition)
- = (for a default)

followed by one or more sub-conditions.

Condition syntax:

    = <sub-cond>
    = <sub-cond> | <sub-cond> ...
    = <sub-cond> & <sub-cond> ...

Sub-conditions are separated by either a \| (meaning or) or a & (meaning
and) allowing us to express some complex logic. It is also possible to
have a combined addition and default conditional by beginning the
conditional with =+ or +=. Two separate conditionals, one addition and
one default, are also permitted preceding a menu entry.

| Sub-condition | Description                                     |
|:--------------|:------------------------------------------------|
| f pattern     | Match current file in the selected panel        |
| F pattern     | Match current file in the unselected panel      |
| d pattern     | Match current directory in the selected panel   |
| D pattern     | Match current directory in the unselected panel |
| t type        | Type of currently selected file                 |
| T type        | Type of last selected file in other panel       |
| x filename    | File is executable                              |
| ! sub-cond    | Negate result of sub-condition                  |

pattern is either a shell pattern (i.e., wildcards) or a regular
expression according to the global setting configured in the
Options/Configuration dialog. This setting can be overridden by adding
shell_patterns=0 as the first line of the menu file. A value of 1 forces
use of shell patterns, while a value of 0 forces regular expressions
instead.

type is one or more of the following: List of file types

| Type | Description      |
|------|------------------|
| r    | regular file     |
| d    | directory        |
| n    | not a directory  |
| l    | link             |
| x    | executable file  |
| t    | tagged           |
| c    | character device |
| b    | block device     |
| f    | FIFO (pipe)      |
| s    | socket           |

To change our image resizing entry to only appear when the currently
selected file has the extension .jpg or .JPG, we would add one line to
the beginning of the entry (regular expressions are used in this
example):

    #   Resize an image using convert

    + f \.jpg$ | f \.JPG$
    R   Resize image to fit within 800 pixel bounding square
        size=800
        convert "%f" -resize ${size}x${size} "%b-${size}.%x"

The conditional begins with + meaning that it’s an addition condition.
It is followed by two sub-conditions. The `|` separating them signifies
an “or” relationship between the two. So, the finished conditional means
“display this entry if the selected file name ends with .jpg or the
selected file name ends with .JPG.”

The default menu file contains many more examples of conditionals. It’s
worth a look.
