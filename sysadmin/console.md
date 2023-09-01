# Console

Most Linuxes, and Ubuntu prior to 18.04:

- 6 linux tty consoles can be accessed with Ctrl+Alt+F1 thru F6.
- F7 is the graphical display.

Ubuntu 18.04 but not MATE

- tty1 is always showing the display manager / login screen.
- tty2 becomes the GUI for the first logged in user.
- tty3-6 available with Ctrl+Alt+F3 thru F6.

## Console Fonts

They're often too small.
To adjust font type, size, etc:

### Ubuntu, Debian:

- Go into a console with Ctrl+Alt+F3
- run: sudo dpkg-reconfigure console-setup
- It runs a menu
- Leave **Encoding** and **Character Set** as is
- Change the font: Terminus is a good choice
- Set the size. Default is 8x16. Way too small on modern hi-res
  displays. Try 16x32
- changes should take place immediately

You can also directly edit (using sudo) /etc/default/console-setup file
and set the font type and size as you wish. For example: font type:
“Terminus Bold” size: 16x32.

    ACTIVE_CONSOLES="/dev/tty[1-6]"
    CHARMAP="UTF-8"
    CODESET="guess"
    FONTFACE="TerminusBold"
    FONTSIZE="16x32"

### Arch-based systems:

List available console fonts.

    ls /usr/share/kbd/consolefonts

Install Terminus fonts if not already installed. Terminus has font sizes larger than 16.

There are a lot of terminus font files, for different code pages, sizes, bold and normal.  
We only care about code page 1.
    ls /usr/share/kbd/consolefonts/ter-1*
    
    ter-112n.psf.gz
    ter-114b.psf.gz
    ter-114n.psf.gz
    ter-116b.psf.gz
    ...etc...

Example:  
Set console font to Terminus, code page 1, size 20, bold
    setfont ter-120b

Make permanent:

edit as root: /etc/vconsole.conf

KEYMAP="us"
FONT="ter-120b"

Displays all characters in the current font
    showconsolefont

