# cron
Cron is "Scheduled Tasks". In cron terminology, a "scheduled task" is a
"cron job"

1.  System job - automatically created by software packages. Runs as
    root (usually).
2.  User jobs - manually created by user. Runs as user (which can be
    root)

In general, **leave the System jobs alone**. Create User jobs.

## Managing User Jobs

Use the **crontab** command:

    crontab -l    # List your cron jobs
    crontab -e    # Edit your cron jobs
    crontab -r    # Remove your cron jobs

Managing root's cron jobs:

    sudo crontab -l
    etc...

root can view any users crontab file by adding "-u username", for
example:

    crontab -u fred -l    # List fred's cron jobs

## Crontab Format

Each line is a collection of six fields. As usual, lines beginning with
pound `#` are comments.

    m     h     dom   mon   dow   command

    *     *     *     *     *     /some/command
    |     |     |     |     |
    |     |     |     |     +----- Day of week (0-7 or Sun Mon Tue Wed Thu Fri Sat)
    |     |     |     +------- Month (1-12 or Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec)
    |     |     +--------- Day of month (1-31)
    |     +----------- Hour (0-23)
    +------------- Minute (0-59)

- An asterisk `*` in a field means all possible values for that field.
- So, 5 asterisks mean "run once a minute, all the time"

Examples:

    # Run a backup script at 2:30 AM every day.
    30  2  *  *  *  /path/to/backup_script

    # Run a command at 4:00 PM on Wednesdays.
    0   4  *  *  3  echo "Hello World."

    # Run a command 'nightly' command at ten minutes past midnight every day
    10  0   *   *   * /bin/nightly

    # Run the 'monday' command every monday at 2 AM
    0   2   *   *   1  /usr/local/bin/monday

Instead of using only single numbers you can use ranges or sets.

    # Use a range of hours matching 1, 2, 3 and 4AM
    *   1-4   *   *   * /bin/some-hourly

    # Use a range with a step to specify every other hour, in the morning
    *   0-11/2  *   *   * /bin/some-every-other-hourly

    # Use step with asterisk to specify every 3 hours, all day
    *   */3  *   *   * /bin/some-every-three-hourly

    # Use a set of hours matching 1, 2, 3 and 5AM
    *   1,2,3,5   *   *   * /bin/some-hourly

Instead of the first five fields, one of eight special strings may
appear:

| string    | meaning                            |
|:----------|------------------------------------|
| @reboot   | Run once, at startup.              |
| @yearly   | Run once a year, `0 0 1 1 *`       |
| @annually | (same as @yearly)                  |
| @monthly  | Run once a month, `0 0 1 * *`      |
| @weekly   | Run once a week, `0 0 * * 0`       |
| @daily    | Run once a day, `0 0 * * *`        |
| @midnight | (same as @daily)                   |
| @hourly   | Run once an hour, `0 * * * *`      |

## Output

**Any** output **(both stdout and stderr)** of the command you run will
be sent to you by email, if your system has a "sendmail" command.  
To stop this, redirect output, as follows:

    0   *   *   *   *  /bin/ls   >/dev/null 2&>1

## Crontab GOTCHAs

Often, something will work from the command line but fail from a
crontab. Most often, the problem is:

1.  wrong crontab notation
2.  environment variables
3.  permissions problem

### Notation

- Crontab uses **24 hour time**, and specifies **minutes first!** So
  "Daily at 11:15PM" in crontab is `15 23 * * *`
- Cron does **not** deal with **seconds** so you can't have cron jobs
  running every 30 seconds.
- Watch out using "day of month" and "day of week" together. This
  becomes an "or" condition not an "and" condition.
- If you want to run a script on Friday the 13th, cron will run the
  script on all Fridays and also the 13th of every month. Your script
  will need to test if it is Friday the 13th.
- Stuff copied from a *system* crontab, won't work as a *user* crontab.
- **System** crontabs have **one extra field** indicating the user. User
  crontabs do not have this field. So you need to remove the extra
  field.
- Stuff copied from other systems using other versions of cron may not
  work. Some crons don't support ranges i.e 4/6
- Crontabs should end with a newline character. This will be assured if
  you use crontab -e
- Do not use a Windows file editor, it will insert newlines with \n\r
  which will not work correctly.
- If your command has any % symbols, it will fail because cron
  interprets % as newline. You need to quote or escape %

### Environment

Crontabs execute in a limited environment, not in your login shell.
Usually the problem is with the PATH. To get around that, just set your
own PATH variable at the top of the script. E.g.

    #!/bin/bash
    PATH=/opt/someApp/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    # rest of script follows

Some prefer to just use absolute paths to all the commands instead.

You can also set the PATH variable in the crontab file, which will apply
to all cron jobs. E.g.

    PATH=/opt/someApp/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    15 1 * * * backupscript --incremental /home /root

Some commands that give status output just won't work in cron, if the
default behavior of a command is to output a line or more to the screen
once the proc starts. For example, *curl* spits out a newline followed
by a progress line. This doesn't work with cron.

So use the silent flag (-s) to tell it not to output any information, or
direct output to /dev/null. For example:

    some-command >/dev/null 

This will redirect only standard output, and not error output (which is
usually what you want, since you want to be informed of errors). To
redirect error output too, use

    some-command >/dev/null 2>&1

### Permissions

- Make sure disk is not full!
- Try executing the command in the shell, as the cron user. Fix any
  problems, then try in cron
- If crontab calls a script, make sure script is executable for the
  crontab's user.
- Make sure the user's password is not expired, or the account is
  inactive.
- If the users account has a crontab but no usable shell in /etc/passwd
  then the cronjob will not run. You will have to give the account a
  shell for the crontab to run.
- If your cronjobs are not running check if the cron daemon is running.
  Then remember to check /etc/cron.allow and /etc/cron.deny files. If
  they exist then the user you want to be able to run jobs must be in
  /etc/cron.allow. You also might want to check if the
  /etc/security/access.conf file exists. You might need to add your user
  in there.
- Cron running at specific times is recorded by default into
  /var/log/syslog. You can `sudo grep cron /var/log/syslog` to look at
  the cron jobs that have run during the current day. You can check
  older days by going through the older log files which are rotated
  daily.

## Cron System Jobs

/etc/crontab  
This is the main system job file. A typical example:

    # /etc/crontab: system-wide crontab
    # Unlike any other crontab you don't have to run the `crontab'
    # command to install the new version when you edit this file
    # and files in /etc/cron.d. These files also have username fields,
    # that none of the other crontabs do.

    SHELL=/bin/sh
    PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

    # m h dom mon dow user    command
    17 *  * * *   root    cd / && run-parts --report /etc/cron.hourly
    25 6  * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
    47 6  * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
    52 6  1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
    #

- run-parts runs all the scripts in each directory. For example, a
  script inside /etc/cron.daily will be executed once per day, every
  day.
- There is also an /etc/cron.d directory. This is for jobs that don't
  fit into the hourly, daily, weekly, monthly schedule.
- anacron is like cron, but does not assume the machine is up
  continuously. This will execute the daily, weekly and monthly jobs
  that got missed because the machine was down.

## Restricting users

Usually, every user can set up schedules. Two configuration files
control who can schedule cron jobs using crontab:

- /etc/cron.allow
- /etc/cron.deny

If /etc/cron.allow exists, only users listed in this file can schedule
cron jobs. If only the /etc/cron.deny file exists, users listed in this
file cannot schedule cron jobs. If neither file exists, any user can
submit cron jobs.

## How cron works

Cron is a daemon that executes scheduled commands. Cron is started
automatically from /etc/init.d on entering multi-user runlevels. Cron
searches its spool area (/var/spool/cron/crontabs) for crontab files
(which are named after accounts in /etc/passwd); crontabs found are
loaded into memory. Note that crontabs in this directory should not be
accessed directly - the crontab command should be used to access and
update them.

Cron also reads /etc/crontab, which is in a slightly different format.
Additionally, cron reads the files in /etc/cron.d.

Cron then wakes up every minute, examining all stored crontabs, checking
each command to see if it should be run in the current minute. When
executing commands, any output is mailed to the owner of the crontab (or
to the user named in the MAILTO environment variable in the crontab, if
such exists).
