# Systemd Cheatsheet

## System state

| Systemd command          | Old command         | Description                             |
|--------------------------|---------------------|-----------------------------------------|
| `systemctl halt`         | `halt`              | Halts the system without turning it off |
| `systemctl poweroff`     | `poweroff`          | Powers off the system                   |
| `systemctl reboot`       | `reboot`            | Restarts the system                     |
| `systemctl suspend`      | `pm-suspend`        | Suspends the system                     |
| `systemctl hibernate`    | `pm-hibernate`      | Hibernates the system                   |
| `systemctl hybrid-sleep` | `pm-suspend-hybrid` | Hibernates and suspend the system       |

You can still use: shutdown; halt; poweroff; and reboot.
Note: halt used to work the same as poweroff in previous Debian releases, but no longer.

## Services

In some of these commands you may get truncated output, use --full for full output.

| Command                             | Description                                                  |
|-------------------------------------|--------------------------------------------------------------|
| `systemctl`                         | List all running services                                    |
| `systemctl start foo.service`       | Activates a service immediately                              |
| `systemctl stop foo.service`        | Deactivates a service immediately                            |
| `systemctl restart foo.service`     | Restarts a service                                           |
| `systemctl reload foo.service`      | Reload configuration without interrupting pending operations |
| `systemctl status foo.service`      | Shows the status of a service                                |
| `systemctl condrestart foo.service` | Restarts if the service is already running                   |


| Command                                    | Description                                      |
|--------------------------------------------|--------------------------------------------------|
| `systemctl list-units --type=service`      | Displays the status of all services              |
| `systemctl list-units --type=mount`        | Displays mounts managed by systemd               |
| `systemctl list-unit-files --type=service` | List the services that can be started or stopped |

| Command                                      | Description                                                    |
|----------------------------------------------|----------------------------------------------------------------|
| `systemctl enable foo.service`               | Start service at next boot                                     |
| `systemctl disable foo.service`              | Service won't be started on next boot                          |
| `systemctl is-enabled foo.service`           | Check if service is already enabled.                           |
| `systemctl is-active foo.service`            | Check if service is running                                    |
| `ls /etc/systemd/system/*.wants/foo.service` | Show which runlevels this service is configured on or off      |
| `systemctl list-dependencies foo.service`    | Show what services need to be running for foo.service to start |

## editing service files
Do NOT directly edit service files. They can get overwritten by a system update.  
Do this instead:

    sudo systemctl edit foo.service

- This creates /etc/systemd/system/foo.service.d/override.conf
- will be preserved when the system is updated.

## Investigate

| Command                                                                             | Notes                                                                               |
|-------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------|
| `systemctl get-default`                                                             | Determine which target unit is used by default.                                     |
| `journalctl -b`                                                                     | Show all messages from last boot.                                                   |
| `journalctl -b -p err`                                                              | Show all messages of priority level *ERROR* and more from last boot.                |
| `journalctl -p warning --since="2015-07-31 12:34:56" --until="2015-09-19 23:59:59"` | View the messages of priority level *WARNING* or more from a certain date and time. |
| `journalctl -f`                                                                     | Follow new messages *a la* tail -f.                                                 |
| `journalctl -u SERVICE`                                                             | Show logs for SERVICE.                                                              |
| `journalctl /usr/sbin/httpd`                                                        | Show all messages related to a specific executable.                                 |
| `journalctl --full`                                                                 | Display all messages without truncating any.                                        |
| `systemctl --state=failed`                                                          | Display the services that failed to start.                                          |
| `systemctl list-units --type=target`                                                | Show current runlevel.                                                              |
| `systemctl isolate graphical.target`                                                | Change the current runlevel (target).                                               |
| `systemctl rescue/emergency`                                                        | Switch to Rescue (single user)/Emergency mode.                                      |
| `systemctl kill SERVICE.service`                                                    | Gently kill SERVICE (*SIGTERM*, 15).                                                |
| `systemd-cgls`                                                                      | Show the full systemd control group (*cgroup*) hierarchy as a tree.                 |
| `systemctl show -p "Wants" multi-user.target`                                       | Find out what other units does a unit depend on.                                    |
| `systemctl list-jobs`                                                               | Show jobs.                                                                          |

## Runlevels

In systemd, runlevels are called *targets*.

| SysVinit runlevel | Systemd target                                        | Notes                                   |
|-------------------|-------------------------------------------------------|-----------------------------------------|
| 0                 | runlevel0.target, poweroff.target                     | Halt the system                         |
| 1, s, single      | runlevel1.target, rescue.target                       | Single user mode                        |
| 2, 4              | runlevel2.target, runlevel4.target, multi-user.target | User-defined runlevels (identical to 3) |
| 3                 | runlevel3.target, multi-user.target                   | Multi-user, non-graphical               |
| 5                 | runlevel5.target, graphical.target                    | Multi-user, graphical                   |
| 6                 | runlevel6.target, reboot.target                       | Reboot                                  |
| emergency         | emergency.target                                      | Emergency shell                         |

Examples:

|                                                                                 |                              |
|:--------------------------------------------------------------------------------|:-----------------------------|
| `systemctl get-default`                                                         | Show current runlevel        |
| `systemctl list-units --type=target`                                            | Show all available runlevels |
| `systemctl isolate multi-user.target` (or) `systemctl isolate runlevel3.target` | switch to runlevel 3         |
| `systemctl isolate graphical.target` (or) `systemctl isolate runlevel5.target`  | switch to runlevel 5         |

Changing the default runlevel:

systemd does not use /etc/inittab file. systemd uses symlinks to point
to the default runlevel.  
The default runlevel in a Debian/Ubuntu system is set in
`/lib/systemd/system/default.target`

    ls -l /lib/systemd/system/default.target
    lrwxrwxrwx 1 root root 16 Apr  3 07:27 /lib/systemd/system/default.target -> graphical.target

The easiest way to change the default runlevel is (as root):

    systemctl set-default multi-user.target

What this does, is create a file, `/etc/systemd/system/default.target`,
that overrides the one in `/lib/systemd/system/default.target` You could
do this manually:

    ln -sf /lib/systemd/system/multi-user.target /etc/systemd/system/default.target


