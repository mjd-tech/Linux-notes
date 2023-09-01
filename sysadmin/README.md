# Administration

## Bash prompt
```
Ctrl-c      kill current process
Ctrl-z      suspend current process, send to background
Ctrl-d      exit the bash shell

Alt-d       delete word (like vim dw)
Ctrl-x Ctrl-e   edit command in $EDITOR. "saving" executes command
Ctrl-l      clear the screen
Ctrl-s      stop output to the screen
Ctrl-q      resume output to the screen
```

## Command line history
```
Ctrl-r      search command history as you type.
            Ctrl-r (again) cycles through the results
            Enter  runs the command
            Tab    puts command on the command line without running
            Ctrl-c goes to a blank command prompt without running
!!          expands to the last typed command. Ex. sudo !!
!$          the last argument of the last command.

```
Note: Install fzf for a much better Ctrl-r experience.

Aggregate history of all terminals in the same .history. On your
  .bashrc:

```bash title=~/.bashrc
shopt -s histappend
export HISTSIZE=100000
export HISTFILESIZE=100000
export HISTCONTROL=ignoredups:erasedups
export PROMPT_COMMAND="history -a;history -c;history -r;$PROMPT_COMMAND"

# double-check all expansions before submitting a command.
shopt -s histverify histreedit
```

- to leave stuff in background even if you logout:


    nohup ./long_script &

- shopt -s cdspell automatically fixes your 'cd folder' spelling
  mistakes
- Add 'set editing-mode vi' in your ~/.inputrc to use the vi keybindings
  for bash and all readline-enabled applications (python, mysql, etc)


