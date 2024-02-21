# Restic Backup Config

https://github.com/restic/restic
https://restic.readthedocs.io/en/stable/

How I set up restic to backup a linux desktop system to a NAS

## Preparation
- Create directory on NAS to receive backups. (restic calls this a "repo").
You need the FULL PATH for the restic config
- ssh shared-key authentication (passwordless) to the NAS
- NAS defined in `/etc/hosts` and `~/.ssh/config`
- In `/etc/fuse.conf`, Uncomment `user_allow_other`
- Create restic directory. `mkdir -p ~/restic{docs,recovery}`

# Installation
- Ubuntu 22.04: Package in repo is out of date. Use package from Debian Sid.
- Download from here https://packages.debian.org/sid/amd64/restic/download
- run `cd Downloads; sudo apt install ./restic_*.deb`

## SetCap
- allows restic to backup system files/directories as normal user, without sudo.
- run `sudo setcap cap_dac_read_search=+ep /usr/bin/restic`
- check it `getcap /usr/bin/restic  # returns nothing if capabilities not set.`
- if you want to remove it `sudo setcap -r /usr/bin/restic`

Caveat:
- When you upgrade the restic package, the restic executable is overwritten
and you lose your setcap settings.
- To fix this, create a "hook" that tells the package manager to setcap restic.

Ubuntu:
- create `/etc/apt/apt.conf.d/50restic-setcap`
- Put this: `DPkg::Post-Invoke { "/usr/sbin/setcap cap_dac_read_search=+ep /usr/bin/restic"; }`

Arch/Manjaro:
- create `/etc/pacman.d/hooks/50-restic-setcap.hook`
- Put this:

```
[Trigger]
Type = Package
Operation = Install
Operation = Upgrade
Target = restic

[Action]
Description = Enable restic to backup entire system without running as root
When = PostTransaction
Exec = /usr/bin/setcap cap_dac_read_search=+ep /usr/bin/restic
```

## Initialize the restic "Repository"

```
restic -r sftp:mynas:/full/path/to/backup/directory init
# It asks for a password: Give it one. Remember it.
```

## Environment variables
create `~/restic/restic-env.sh`

```
export RESTIC_REPOSITORY=sftp:mynas:/full/path/to/restic/repository/
export RESTIC_PASSWORD=password
```

Put the following in your .bashrc and/or .zshrc.
```
# restic environment variables
source "$HOME/restic/restic-env.sh"
```
Also put this in any shell scripts you create for restic.

## Dry run backup /etc
Look for "Would add to the repository: ..."
```
restic backup --dry-run /etc
```

## Next steps
- backup script
- excludes file
- cron
- recovery

