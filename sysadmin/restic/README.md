# Restic Backup Configuration
How I set up restic to backup a linux desktop system to a NAS

## Preparation
- Create directory on NAS to receive backups. (restic calls this a "repo").  
Take note of the FULL PATH. You need it for restic configuration
- Ensure ssh shared-key authentication (passwordless) to the NAS (see below)
- In `/etc/fuse.conf`, Uncomment `user_allow_other` (need sudo to do this)
- Download this git repo as a zip file, extract the restic directory, and put it in your home directory.
- should result in:
```
/home/your_username/restic
├── docs
│  ├── cmds.txt
│  ├── cron.txt
│  ├── laptops.md
│  ├── prune-snapshots.snippet
│  └── README.md
├── jobs
│  ├── backup
│  ├── backup.excludes
│  └── sysinfo
├── repos
│  └── myrepo
├── restic-mountpoint
├── Restic-ui
└── README.md
```
- make shell scripts executable
```bash
chmod +x ~/restic/Restic-ui
chmod +x ~/restic/jobs/backup
chmod +x ~/restic/jobs/sysinfo
```

## Install restic
- See the README file in the docs folder

## Initialize the restic "Repository"
```bash
restic -r sftp:mynas:/full/path/to/backup/directory init
# It asks for a password: Give it one. Remember it.
```

## Run Restic-ui
- Here, you can modify jobs, repository configs, set cron jobs, run restic commands, etc
- NOTE: files in the "jobs" and "repos" folder contain placeholders that **must be changed** to your needs.

## Dry run backup /etc
Look for "Would add to the repository: ..."
```bash
source ~/restic/repos/myrepo  # or whatever your repo is named
restic backup --dry-run /etc
```
### SSH notes
In case ssh "shared-key" (passwordless) authentication is not configured
and needs to be set up from scratch.

- ensure NAS is in `/etc/hosts`
```
mynas 192.168.1.10
```
test it with `ping -c4 mynas`. should get 4 sucessful pings.

create/edit `~/.ssh/config` file
```
Host        mynas
HostName    mynas
Port        22
User        fred
```
test it with `ssh mynas`. should ask for password, it wants the NAS password

- set up ssh keys, copy to NAS.

```bash
ssh-keygen -c "SOMETHING"
# Replace SOMETHING with your name or initials or email address

# Copy your public key to the remote host:
ssh-copy-id -i ~/.ssh/id_rsa.pub mynas

# test it
ssh mynas       # should put you into ssh session with no password prompt
```
### Running sudo commands in shell scripts
**Method 1:**
- create `/etc/sudoers.d/myconf`
- Put this: `%sudo ALL = NOPASSWD: /usr/bin/some_cmd arg1 arg2...`
- args are optional
- it's `%wheel` in Arch and Fedora based systems

**Method 2:**
> This method feeds the sudo password to the command.  
> But we don't want the password to be in clear text, so we use base64 encoding.

In the terminal
- type a `space` then `echo yourpassword | base64 > ~/.config/hash`
- NOTE: leading space prevents command (and password) appearing in your `~/.bash_history`

In your script
- put: `base64 -d < ~/.config/hash | sudo -S true 2>dev/null`
- this runs the command "true", which does nothing.
- But it "activates" sudo for further use without a password
- now you can use one or more `sudo some_cmd` in the script

**NOTE:** If you change your password, need to **re-generate** the "hash" file

### getting email notification
- email is sent by cron, not restic
- you need an email "MTA" such as msmtp or Postfix.
- The backup script is designed to work both with and without an MTA

## Reference
- https://github.com/restic/restic
- https://restic.readthedocs.io/en/stable/