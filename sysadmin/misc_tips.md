# Miscellaneous Tips

## Deleting Multiple Files

If you get this error: `/bin/rm: Argument list too long.`
use the find command with its built-in "-delete" option:

```bash
find . -type f -delete

# show the filenames as you’re deleting them:
find . -type f -print -delete

```

## logout but leave task running:

```bash
nohup ./long_script &
```

## Disable Guest Account

This is for Ubuntu desktops that use lightdm

## For Ubuntu 16.04 LTS (15.10 or later)

Just run this (once) at terminal:

    sudo sh -c 'printf "[Seat:*]\nallow-guest=false\n" >/etc/lightdm/lightdm.conf.d/50-no-guest.conf'

To undo (restore Guest option), remove the file created:

    sudo rm /etc/lightdm/lightdm.conf.d/50-no-guest.conf

## Hide users from login screen

- Ubuntu 16.04 uses Accounts Service.
- You can not hide a user from the greeter screen by reconfiguring
  lightdm because it defers to AccountsService.
- That is stated very clearly in the comments in
  /etc/lightdm/users.conf.
- What you need to do instead is to reconfigure AccountsService.

To hide a user named XXX, create a file named

    /var/lib/AccountsService/users/XXX

containing two lines:

    [User]
    SystemAccount=true

If the file already exists, make sure you append the SystemAccount=true
line to the \[User\] section.


- Aggregate history of all terminals in the same .history. On your
  .bashrc:

    shopt -s histappend
    export HISTSIZE=100000
    export HISTFILESIZE=100000
    export HISTCONTROL=ignoredups:erasedups
    export PROMPT_COMMAND="history -a;history -c;history -r;$PROMPT_COMMAND"

- Pressed 'Ctrl-s' by accident and the terminal is frozen? Unfreeze:
  'Ctrl-Q'

## Unix buffering ruins your day

You write some shell command like:

    tail -f /var/log/foo | egrep -v 'some|stuff'

and wonder why nothing is printed, even though you know some text has
matched. The problem is that stdout is being buffered.

By default, writes to stdout pass through a 4096 byte buffer, unless stdout
is a terminal/tty, in which case it is line buffered.

Hence, the inconsistency between the immediate output when your program
is writing to the terminal and the delayed output when it is writing to
a pipe or file.

One solution is to use `stdbuf`, part of GNU coreutils.
```bash
tail -f /var/log/foo | stdbuf -o0 egrep -v 'some|stuff'
```

This doesn't always work. For example: mawk, (the default awk in Debian/Ubuntu)
does not seem to work with stdbuf. It does however provide a -Winteractive option
which will turn off buffering.
```bash
tail -f /var/log/foo | mawk -Winteractive
```

GNU sed provides the -u option providing unbuffered output. 
You can also use stdbuf as above.

```bash
tail -f /var/log/foo | sed -u

# or
tail -f /var/log/foo | stdbuf -o0 sed
```

GNU grep provides  --line-buffered, to disable buffering,
or again you can use stdbuf.

```bash
tail -f /var/log/foo | grep --line-buffered

# or
tail -f /var/log/foo | stdbuf -o0 grep
```
