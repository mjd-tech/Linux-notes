# Restore
Two ways
1. restic restore - to restore many files
2. restic mount - easiest way to restore just a few files

## restic restore
- get list of snapshots with id numbers `restic snapshots`
- restore a directory `restic restore 79766175:/home/fred/.thunderbird --target /home/fred/tmp`

## restic mount
- Mount the restic repository `restic mount --allow-other ~/restic/recovery`
- use gui file manager to drag and drop.
- for files/directories owned by root, right click...Open as Administrator
- or use midnight commander as root `sudo mc`

## Script to mount/unmount restic repository
- can run on same or another box (provided restic is installed).  
- Modify as needed.

```bash
#!/bin/bash
# mount or unmount a restic repository

# shellcheck disable=2120,2016

########## VARIABLES
script_dir="$(dirname "$(readlink -f "${BASH_SOURCE[0]}")")"
mount_dir="$script_dir/recovery"
if [[ -f $script_dir/vars.sh ]]; then
    source "$script_dir/vars.sh"
else
    # SET THESE IF YOU DON'T HAVE a vars.sh file
    export RESTIC_REPOSITORY=sftp:user@192.168.x.x:/full/path/to/restic/repository
    export RESTIC_PASSWORD=password
fi
########## END VARIABLES

# Bash - set exit code to non-zero if any command in pipeline fails
set -o pipefail

# text colors
rst='\e[0m'; yel='\e[1;33m'; grn='\e[1;32m'; red='\e[1;31m'

########## FUNCTIONS

Die () {
    echo -e "\n${red}ERROR:${rst} ${1:-}" >&2
    echo -e "\nPress any key to exit..."
    read -rsn1; exit 1
}

Ask () {
    while true; do
        echo -e "\n$1 [y/n]:"
        read -rsn1
        case $REPLY in
            [Yy])   return 0 ;;
            [Nn])   return 1 ;;
            *)      echo "Please answer y or n"
        esac
    done
}

Pause () {
    echo -en "\n${grn}Press any key to ${1:-continue}...${rst}"
    read -rsn1
    # clear to: end of line, beginning of line; move cursor to beginning of line
    tput el; tput el1; echo -en "\r"
}
########## END FUNCTIONS

# Create mount point if needed
if ! [[ -d $mount_dir ]]; then
    mkdir -p "$mount_dir" || Die "Cannot create $mount_dir"
fi

# Check if repository already mounted
mountpoint -q "$mount_dir" && {
    echo -e "Repository is mounted to ${yel}$mount_dir${rst}"
    if Ask "Unmount the repository?"; then
        fusermount -u "$mount_dir" || Die "Could not unmount the repository"
        echo; echo "Repository unmounted OK"
        Pause "exit"; exit
    else
        Pause "exit"; exit
    fi
}

# Mount the repository
echo -e "Attempting to mount restic repository to ${yel}$mount_dir${rst}"

restic mount --allow-other "$mount_dir" &>/dev/null &
for (( i=0; i<10; i++ )); do    # wait for repository to mount
    echo -n "."
    [[ -d $mount_dir/snapshots ]] && break
    sleep 1
done
[[ -d $mount_dir/snapshots ]] || Die "Could not mount restic repository"

# Display mount info
echo -e "\nRepository is mounted (read-only) at ${yel}$mount_dir${rst}"
echo -e "\nRemember to ${yel}UNMOUNT${rst} the repository when you are finished!"
Pause
#trap 'mountpoint -q "$mount_dir" && fusermount -q -u "$mount_dir" &>/dev/null' EXIT
```