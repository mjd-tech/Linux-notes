# Terminal - Alternate screen

## The Problem

- View a file with `less`, find the section of interest, exit `less`,
  the screen clears.
- You wanted it to remain on the screen.
- Particularly annoying with man pages. `man` uses `less` to display.
- vim also has this feature, but in practice it is not as annoying.

The underlying feature is called "alternate screen"

- It's a feature of your terminal (be it xterm, gnome-term or even your
  console).
- By default `less` uses the alternate screen for display.
- Upon exit, less tells the terminal to switch back to the normal
  screen.

## Easy solution

In your .bashrc, put this

    alias less="less -X"
    export MANPAGER="less -Xs"

- The extra "s" option is needed for MANPAGE
- By default, man uses `pager -s`
- This squeezes consecutive blank lines into a single blank line.

## Downside

- The terminal's scrollback buffer gets filled with the file or man page
  contents
- This is similar to what happens if you use `cat`.

Remember you can use `head` or `tail` instead of `less`. You can also
use `head` and `tail` for man pages:

    man grep | head

For vim, you can put this in your .vimrc

    set t_ti= t_te=

However, this has the tendency to "clobber" your scrollback buffer,
leaving only the vim session on the screen.
