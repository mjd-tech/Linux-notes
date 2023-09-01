# cd
Change directory. This command is built into the Bash shell, so there is
no separate manual page. Try `help cd` or `man bash`. This command is
pretty simple anyway.

    cd          # change directory to your home directory
    cd ~        # same
    cd /etc     # change to specific directory, in this case /etc
    cd -        # change to last directory
    cd ..       # go up one level
    cd ../..    # go up two levels

## pushd and popd
- use the "directory stack" as a form of "bookmarks".
- Clumsy to use interactively, but occasionally useful in scripts.

## zoxide
zoxide is a "smarter cd" command.
It remembers which directories you use most frequently,
so you can "jump" to them in just a few keystrokes.

Install zoxide and fzf (v0.33.0 and above). The versions in Ubuntu repos tend to be old.

```
z foo              # cd into highest ranked directory matching foo
z foo bar          # cd into highest ranked directory matching foo and bar
z foo /            # cd into a subdirectory starting with foo

z ~/foo            # z also works like a regular cd command
z foo/             # cd into relative path
z ..               # cd one level up
z -                # cd into previous directory

zi                 # cd with interactive selection (using fzf)
zi foo             # cd with interactive selection (using fzf) with foo as query string.

z foo<SPACE><TAB>  # show interactive completions (zoxide v0.8.0+, bash 4.4+/fish/zsh only)

zoxide edit        # edit the bookmarks (using fzf as interactive interface)
```




