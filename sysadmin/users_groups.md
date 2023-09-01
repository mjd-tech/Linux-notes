# Users and Groups
Users and groups are used to control access to files, directories, and peripherals.

## Examples
Which groups am I in?
    groups

What's my user ID
    id  # also displays group info

Add fred to vboxusers group
    sudo usermod -G vboxusers -a fred

Add new user, Debian-based systems. Runs an interactive "wizard"
    sudo adduser barney

Add new user, Arch-based systems
    # Creates and populates homedir, shell = bash, primary group same name as user
    sudo useradd barney
    
    # With fullname, shell =zsh, add to vboxusers and wheel groups.
    sudo useradd -m -c "Barney Rubble" -s /bin/zsh -G vboxusers,wheel barney
    
    # Set password for new user (recommended)
    sudo passwd barney

Add new system user "foo", no homedir, no login
    sudo useradd --system --shell=/usr/sbin/nologin foo
    # shell may be /usr/bin/nologin. check /etc/password for examples

Delete user
    sudo userdel -r barney

## Primary group
- Typically, when you create new user barney, a new group named barney is created.
- This group becomes the "primary group" for barney
- files created by barney are owned by barney:barney
- On a NAS, typically all users primary group is "users", to facilitate file sharing.

## files
| File | Description |
|:---- |:---|
| /etc/password     | user database |
| /etc/shadow       | password database |
| /etc/group        | group database |
| /etc/default/useradd | defaults for useradd command |
| /etc/login.defs   | more defaults for useradd, and things like password aging |
| /etc/skel/.*      | dotfiles for new user's home directory

## fields in /etc/passwd
Fields are separated by colon :

| Field       | Description                                                     |
|:----------- |:--------------------------------------------------------------- |
| login name  | example: barney                                                 |
| password    | "x" - the encrypted password is stored in /etc/shadow           |
| uid         | user id number identifying each user                            |
| gid         | user's primary group id number                                  |
| GECOS       | data field usually containing the user's full name              |
| homedir     | user's home directory                                           |
| login shell | typically /bin/bash, /bin/nologin, /usr/bin/nologin, /bin/false |

## fields in /etc/group

| Field      | Description                                          |
| ---------- | ---------------------------------------------------- |
| group name | name of group                                        |
| password   | usually "x".  group passwords are almost never used. |
| gid        | group id number                                      |
| members    | comma delimited list of group members                |

## fields in /etc/shadow
- username
- encrypted password
- several fields managing password expiration.

## useradd vs adduser
useradd is a standard command found on any Linux system.

adduser is found on Debian, Ubuntu and derivatives.
- It's an interactive script.
- It's a "wrapper" for the useradd command
- Debian also includes scripts: deluser, addgroup, delgroup, etc.

## Other Commands
| Command    | Description                                            |
|:---------- |:------------------------------------------------------ |
| `chfn`     | change fullname                                        |
| `chsh`     | change shell. /bin/false disables an account           |
| `usermod`  | change UID, home directory, lock/unlock password, etc. |
| `userdel`  | optionally delete home directory and mail spool        |
| `groupadd` | creates a new group with no users                      |
| `groupdel` | delete group                                           |                                                         |

## getent
getent (get entries) command queries system databases.

Instead of cat /etc/passwd
    getent passw

Instead of grep fred /etc/passwd
    getent passwd fred

Instead of cat/etc/group
   getent group
