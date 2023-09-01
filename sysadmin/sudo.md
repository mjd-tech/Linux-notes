# Sudo
Run command as root or  optionally another user.

By default
- sudo prompts you for your password
- sets a timer for **15 minutes**, where it won't prompt you again.

## Run sudo without password

While you could edit /etc/sudoers, it's better to **create a file in /etc/sudoers.d**

    sudo visudo -f /etc/sudoers.d/myconf

Allow "admin" user(s) to use sudo without password:

    # Debian, Ubuntu and derivatives
    %sudo ALL = NOPASSWD: ALL
    # Arch Linux, Red Hat and derivatives
    %wheel ALL = NOPASSWD: ALL


## Run a specific command without password
Allow "admin" user(s) to run "rsync" as root, without password.
This is useful for cron job scripts, where you can't interactively enter the password.

    %sudo ALL = NOPASSWD: /usr/bin/rsync

## Changing the timeout

The default timeout is 15 minutes. To change it:

    sudo visudo -f /etc/sudoers.d/myconf

Add one of the following:

    # Change timeout to 1 hour
    Defaults    timestamp_timeout=60

    # Require password once. Timeout lasts as long as shell session.
    Defaults    timestamp_timeout=-1

- Set timestamp_timeout to whatever time you want in minutes
- Set to 0 if you want a password prompt for every sudo command
- Set to -1 if you want timestamp to last as long as your bash session

## Prompt for Password Once Per System Boot

To make sudo prompt for a password once per system boot, add the
following entries in the /etc/sudoers file:

    sudo visudo -f /etc/sudoers.d/myconf

add the following

    Defaults    timestamp_timeout=-1
    Defaults    timestamp_type=global
    # Old way:  Defaults   !tty_tickets

## Sudo Options

| Option   | Description                                                            |
|:---------|:-----------------------------------------------------------------------|
| -k       | force the timer to time out now                                        |
| -l       | list the permissions of the sudo invoking user                         |
| -u       | run command as user other than root                                    |
| -s       | run a root shell, best used with -H                                    |
| -H       | run using root's "environment" but leaves you in the current directory |
| -i       | like -sH but puts you in /root directory                               |
| -e       | edit one or more files instead of executing a command                  |
| sudoedit | is an alias for "sudo -e"                                              |

### Sudoers file

Note:

- Sudo was designed for "Enterprise" use.
- You'd have **one sudoers file** used on **all your hosts**.
- This is rarely done in practice.

The basic format is 4 fields:

    user host = (runas) command

Read this as: **User** may run **Command** as the **Runas** user on **Host**

The special keyword **ALL** can be used to indicate "any user" "any host" "any command"

**User** field

- Linux user: ex. **fred**
- Linux group prefixed with %: ex. **%sudo**
- Comma separated list: ex. **fred,barney,%jetsons**

Most of the time you will use %sudo (Debian,Ubuntu) or %wheel (OtherLinuxes) group here.

**Host**

- Use **ALL** here.
- You could use the hostname here, but it's pointless. Just use ALL

**Runas** is optional, must be enclosed in parentheses ()

- If omitted, the command(s) run only as root.
- You can also specify the group that the command runs as e.g. (postgres:postgres)
- This is why you sometimes see (ALL:ALL)

**Command** is

- **Full path** to an executable.
- You can include arguments to the executable, to permit the command only with those exact arguments.
- Or, put "" to only permit execution without arguments.
- A command may also be the full path to a directory (including a trailing /). 
This permits execution of all the files in that directory, but not in any subdirectories.

**Command Options**

- Before the command, you can specify zero or more options to control how it will be executed.
- Most often used is NOPASSWD (to not require a password)
  
    someuser ALL=(ALL) NOPASSWD: /bin/ls

**Secure Path**

- In /etc/sudoers there is a "secure_path" setting.
- This is like your shell's PATH variable
- If you reference a command that's not in the secure path, it won't work

## Sudo password in script

    echo 'password' | sudo -S command

> :warning: This method should be considered **extremely insecure**  
> The password is stored in **plain text**

The -S option means "read password from standard input"

To make it slightly less insecure, **obfuscate** the password.

    # prevent this command from being stored in history by starting with a space
     echo "yourpassword" | base64 > ~/home/.config/foo

In your script:

    base64 -d < $HOME/.config/foo | sudo -S somecommand

## Locked out of sudo
If you have 3 failed sudo attempts, you're locked out of sudo for 10 minutes.
fix it: `faillock --user fred --reset`

