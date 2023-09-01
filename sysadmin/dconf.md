# Dconf Database

- Holds settings for Mate desktop environment and **some** applications
- Database file `$HOME/.config/dconf/user`
- It's a **binary** file. Can access with `dconf` (cli tool) also `dconf-editor` (gui)
- Everything is in "Reverse domain name notation" ie /org/mate/panel

## Dump database to text file
This is the preferred way to backup the dconf database.

    # Entire Database:
    dconf dump / > dconf_all.txt
    # Mate Panel
    dconf dump /org/mate/panel/ > dconf_mate_panel.txt

## Load database from text file

    # Entire Database:
    dconf load / < dconf_all.txt
    # Mate Panel
    dconf load /org/mate/panel/ < dconf_mate_panel.txt

## Dump from a backed up binary "user" file
- Only do this if you don't have a text backup.
- You have to create a "profile".

    # copy and rename the binary database file
    cp /path/to/binary/user ~/.config/dconf/user.bak

    # make the "profile" in your home directory
    cd
    echo "user-db:user.bak" > ~/prof

    # Dump all
    DCONF_PROFILE=~/prof dconf dump / > ~/old_settings

    # Dump Mate Panel
    DCONF_PROFILE=~/prof dconf dump /org/mate/panel/ > mate_panel_settings
