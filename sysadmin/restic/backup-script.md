# Backup Script
- intended to be a "daily backup", run by cron at "wee hours of the morning"
- backs up the entire system, except what's in `excludes.txt`
- writes status info to a log file, which is auto-truncated, so it doesn't get huge.
- You can optionally configure an email sender like `msmtp` and receive status via email.

- save the script below to `~/restic/backup`
- paste in optional code
- Replace all **CHANGEME** with correct info

```bash
#!/bin/bash
# Backup script for restic

# shellcheck disable=2120,2016

########## VARIABLES
backup_host=CHANGEME

script_dir="$(dirname "$(readlink -f "${BASH_SOURCE[0]}")")"
source "$script_dir/vars.sh"
log_file="$script_dir/backup.log"
excludes="$script_dir/excludes.txt"
########## END VARIABLES

# Bash - set exit code to non-zero if any command in pipeline fails
set -o pipefail

# Functions used in this script.
LogIt () { echo "$@" | tee -a "$log_file"; }
PingIt () { ping -qc 3 "$backup_host" &>/dev/null; }

# Truncate log file
tmp=$(tail -n 200 "$log_file" 2>/dev/null) && echo "$tmp" > "$log_file"

# Initialize logging
LogIt
LogIt "======================== $(date '+%D %r') ========================"

### Optional code goes here

### End optional code

# Make sure we can ping the remote host before attempting backup
PingIt || { LogIt "ping $backup_host FAILED"; exit 1; }

# Do the backup
LogIt
LogIt "Running backup..."
restic backup --exclude-file="$excludes" / |& tee -a "$log_file" || {
    LogIt "Backup failed"
    exit 1
}
LogIt "OK"

# Remove old snapshots
LogIt
LogIt "Removing old snapshots..."
restic forget \
    --keep-daily 7 --keep-weekly 4 --keep-monthly 12 \
    --prune |& tee -a "$log_file" || {
    LogIt "Removing old snapshots failed"
    exit 1
}
LogIt "OK"
```
## Optional code

### Wake up network connection
```bash
## Wake up network connection if needed. Useful for wifi connections
net_connection="CHANGEME"       # Network Manager connection name
LogIt "Checking Network connection: $net_connection ..."

if ! PingIt; then
    nmcli con up "$net_connection" || {
        LogIt "Network connection: $net_connection FAILED"
        exit 1
    }
    sleep 3
    # try again
    PingIt || { LogIt "ping $backup_host FAILED"; exit 1; }
    sleep 3
fi
LogIt "OK"
```
### Installed Packages - Debian/Ubuntu
```bash
## Installed Packages - Debian/Ubuntu
LogIt
LogIt "Getting list of all installed packages..."
dpkg --get-selections > "$HOME/restic/all-packages.txt" || exit 1
LogIt "OK"
LogIt "Getting list of manually installed packages..."
comm -23 <(apt-mark showmanual | sort -u) \
         <(gzip -dc /var/log/installer/initial-status.gz \
             | sed -n 's/^Package: //p' | sort -u) \
         > "$script_dir/my-packages.txt" || exit 1
LogIt "OK"
```

### Installed Packages - Arch/Manjaro
```bash
## Installed Packages - Arch/Manjaro
LogIt
LogIt "Getting list of explicitly installed packages..."
pacman -Qqen > "$script_dir/pkglist.txt" || exit 1
pacman -Qqem > "$$script_dir/pkglist_aur.txt" || exit 1
LogIt "OK"
```
### Flatpaks
```bash
## Flatpaks
LogIt
LogIt "Getting list of Flatpaks..."
flatpak list --app > "$script_dir/flatpaks.txt" || exit 1
```

### dconf database
```bash
## Dump dconf database to text file
LogIt
LogIt "Dumping dconf database to text file..."
dconf dump / > "$script_dir/dconf_settings.txt" || exit 1
LogIt "OK"
```

### Running sudo commands in the backup script
Two ways to do this
1. configure "sudoers" to allow executing the command without a password
2. pipe the password to sudo

Method 1:
- create `/etc/sudoers.d/myconf`
- Put this: `%sudo ALL = NOPASSWD: /usr/bin/some_cmd arg1 arg2...`
- args are optional
- it's `%wheel` in Arch and Fedora based systems

Method 2:
To avoid having the password in clear text, use base64 encoding.
- in terminal, type a `space` then `echo "yourpassword | base64 > ~/.config/hash`
- leading space prevents this command (and password) appearing in your `~/.bash_history`
- in your script: `base64 -d < ~/.config/hash | sudo -S true 2>dev/null`
- this runs the command "true", which does nothing.
- But it "activates" sudo for further use without a password
- now you can use one or more `sudo some_cmd` in the script
- can use this technique in other scripts
- if you change your password, need to re-generate the "hash" file

Some sudo commands:
```bash
# Clean apt cache
LogIt
LogIt "Cleaning apt cache"
sudo apt-get clean || exit 1
LogIt "OK"

# Trim systemd journal
LogIt
LogIt "Vacuuming systemd journal"
sudo journalctl --vacuum-time=2weeks || exit 1
LogIt "OK"
```