# Misc Terminal Utilities

Note: Some of these are in the Ubuntu repo, but are out of date

## bat

- enhanced "cat" with syntax highlighting, including man pages
- <https://github.com/sharkdp/bat>
- Version in Ubuntu repo does not format man pages properly.
- Also, this version installs as batcat, to avoid conflict with "bacula"
  (backup program).
- I don't use Bacula so I don't need this.
- <https://github.com/sharkdp/bat/releases/download/v0.18.3/bat_0.18.3_i686.deb>
- bat can be used as a colorizing pager for man, by setting the MANPAGER
  environment variable:

<!-- -->

    # Put this in your .bashrc
    export MANPAGER="sh -c 'col -bx | bat -l man -p'"
    # if you experience formatting problems,
    export MANROFFOPT="-c"

## ripgrep (rg)

- Enhanced "grep"
- By default, recursively searches files in current directory for a
  regex pattern
- By default, ripgrep respects gitignore rules (when within a git repo) and automatically skips
  hidden files/directories and binary files.
- Supports searching compressed files with the -z --search-zip flag.
- Can also act like "sed" with the -r --replace flag
- Better regex engine than grep
- Much faster than grep or other tools such as ack.
- <https://github.com/BurntSushi/ripgrep>
- <https://github.com/BurntSushi/ripgrep/releases/download/13.0.0/ripgrep_13.0.0_amd64.deb>

Ripgrep and Bat conflict in 20.04, fix with

    sudo apt install ripgrep
    sudo sed -i '/\/usr\/.crates2.json/d' /var/lib/dpkg/info/ripgrep.list
    sudo apt install bat
    sudo ln -s batcat /usr/bin/bat
