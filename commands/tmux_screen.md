# tmux and GNU Screen
multiplexer for terminal sessions, with detach.

tmux is nicer out of the box than screen.

Useful with ssh connections to remote hosts.
- Multiple terminal "windows" over a single ssh connection.
- "Detach" from long-running processes such as upgrades or compiling.
- screen can also be used as a "comm" program for serial ports.

However tmux and screen both have clumsy scrolling and copy/paste operations.  

## Use on remote host
tmux must be installed on the remote host.  
- ssh into the host and run `tmux`
- start a long-running task
- detach with CTRL-b d
- this also disconnects the SSH session, but the task is still running

To reattach:
- ssh into the host and run `tmux attach`

Note: if you somehow created multiple tmux sessions:
    tmux ls     # list sessions
    tmux attach-session -t 0  # attaches to session 0

## Tmux Key Bindings
All keybindings begin with **Ctrl-b**

    Most used commands:
    ?   Shows Help
    c   Create new window
    n   Go to next window
    p   Go to previous window
    0-9 Go to window 0-9
    w   list windows
    &   close window

    [   Enter copy mode
          Space   Start selection
            use arrow keys or vim motion keys to select
          Enter   Copy selection
          q 	  Quit copy mode
    
    ] 	Paste contents of buffer_0
    
    d   detach

    Send a literal Ctrl-b to the application:
    Ctrl-b Ctrl-b    For example, Vim uses Ctrl-b to "go up one screen"

## Screen as "comm" program

This example is for the Sheeva Plug which has a usb console connection

    screen /dev/ttyUSB1 115200

Note: You need to be a member of the "dialout" group, or else run this
command with sudo.  
You can add yourself to the dialout group like this

    sudo adduser $USER dialout

You need to **log out and log back in** for group membership to take
effect.
