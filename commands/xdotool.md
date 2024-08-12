# xdotool
Send keystrokes to a window

Example: Launch mate-terminal and send keystrokes with xdotool

```bash
#!/bin/bash
mate-terminal & sleep 0.2
xdotool type "echo foo"
xdotool key "Return"
```

- You need the sleep, otherwise it sends keystrokes before mate-terminal is ready.
- NOTE: use & not &&
- mate-terminal remains running after the script ends,
  and it is the "active window" that has the "focus"

Another way is use xdotool's builtin sleep command
```bash
#!/bin/bash
mate-terminal &
xdotool sleep 0.2 type "echo foo"
xdotool key "Return"
```

To run this from the command line, you need a one-liner.
```bash
mate-terminal & xdotool sleep 0.2 type "echo foo"; xdotool key "Return"
```

If you try to enter the commands one at a time, the new terminal launches and becomes the "active window".
If you keep typing, it goes in the new terminal. Not what you want.
So you go back to the original terminal and run the xdotool commands.
But now the original terminal is the "active window".
You are sending keystrokes to yourself, not the new terminal.
The one-liner prevents this scenario.

Example:
launch mate terminal, run a command, copy the output to clipboard, then launch pluma, paste.

```bash
#!/bin/bash

# launch mate-terminal, which becomes the "active window"
mate-terminal & sleep 0.2             # wait for terminal to fully launch
xdotool type 'cat ~/.ssh/config'      # send the command
xdotool key 'Return'                  # send the Enter key
sleep 0.2                             # wait for command to finish
xdotool key ctrl+shift+a              # Select all
xdotool key ctrl+shift+c              # Copy to clipboard

# launch pluma, which becomes the "active window"
pluma & sleep 0.2                     # wait for pluma to fully launch
xdotool key ctrl+v                    # Paste from clipboard
```

xdotool works with the "active window"
Sometimes you need to work with an "inactive window"

Reverse the order of the above example:
launch pluma, launch mate-terminal, run command, copy the output go back to pluma, paste
```bash
  #!/bin/bash

  # launch pluma, which becomes the "active window"
  pluma & sleep 0.2                     # wait for pluma to fully launch
  wid=$(xdotool getactivewindow)        # get window id

  # launch mate-terminal, which becomes the "active window"
  mate-terminal & sleep 0.2             # wait for terminal to fully launch
  xdotool type 'cat ~/.ssh/config'      # send the command
  xdotool key 'Return'                  # send the Enter key
  sleep 0.2                             # wait for command to finish
  xdotool key ctrl+shift+a              # Select all
  xdotool key ctrl+shift+c              # Copy to clipboard

  # Switch to pluma
  xdotool windowactivate $wid           # Activate the pluma window
  sleep 0.2                             # wait for window to fully activate
  xdotool key ctrl+v                    # Paste from clipboard
```

There's a lot of sleep going on. It's there for a reason.
Without sleep, xdotool will gladly send keystrokes to the wrong window, without throwing an error.

xdotool can also
- minimize/maximize windows
- resize windows
- move windows
- put windows in another workspace (probably the most useful)
- mouse moves and clicks (difficult to do anything useful. fragile)

xdotool windowmove [options] [window] x y


The main shortcoming with xdotool is getting the right window id especially if there's multiple instances running already.
It can be done, sort of, but there's too many "landmines"

So xdotool is best used when your script lauches a gui app
and you immediately do your xdotool stuff.

Example: launch some apps and send them to desktop 2
```bash
#!/bin/bash
gvim & sleep 0.2
xdotool getactivewindow set_desktop_for_window 1      # Desktops are zero based

mate-terminal & sleep 0.2
xdotool getactivewindow set_desktop_for_window 1      # Desktops are zero based
```

For some reason, you have to getactivewindow before set_desktop_for_window.
You don't have to do this the with the "type" or "key" command.
One of many annoying inconsistencies.
That and having to use sleep all over the place.
There's no mention in the man pages that unless you use sleep nothing works.

If you're going to do anything more involved than these examples, you're in for a world of pain.

There is a complication when using xdotool with other tools for window handling:
xdotool uses decimal numbers for windwow ids,
most other tools use hexadecimal numbers.

For example, if you find a window with:
  xdotool getactivewindow,
you will not find the result in the output of xwininfo -root -tree, that lists all windows.
It needs to be converted to a hexadecimal number first:

```bash
$ xdotool getactivewindow
69206716

$ printf 0x%x 69206716
0x42002bc

$ xwininfo -root -tree | grep 0x42002bc
           0x42002bc (has no name): ("konsole" "Konsole")  1154x781+0+0  +1289+498

Converting decimal to hexadecimal:
  printf 0x%x 69206716

Converting hexadecimal to decimal:
  printf %i 0x42002bc
```
