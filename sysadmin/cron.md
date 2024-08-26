# cron
Cron is how Linux does "Scheduled Tasks".  
You configure "cron jobs" using the `crontab` command.  
There are two types of cron jobs:

1.  System jobs - automatically created by software packages. Runs as root (usually).
2.  User jobs - manually created by user. Runs as user (which can be root)

In general, **leave the System jobs alone**. Create User jobs.

## Managing User Jobs

Use the **crontab** command:

```bash
crontab -l    # List your cron jobs
crontab -e    # Edit your cron jobs
crontab -r    # Remove your cron jobs

sudo crontab -l    # List root's cron jobs
sudo crontab -e    # Edit root's cron jobs
sudo crontab -r    # Remove root's cron jobs
```

root can view any users crontab file by adding "-u username"
```bash
sudo crontab -u fred -l    # List fred's cron jobs
```
## Examples
```bash
# Daily at 3:30AM (cron uses 24 hour time)
30 3 * * *      /path/to/command

# Daily at 12:00AM (using a special string)
@daily          /path/to/command

# First day of the month at 7:00PM (cron uses 24 hour time)
0 19 1 * *      /path/to/command

# Tuesdays and Fridays at 4:30AM (using a "list")
30 4 * * Tue,Fri    /path/to/command

# Monday thru Friday at 11:45PM (using a "range")
45 23 * * Mon-Fri   /path/to/command

## Every 4 hours (using a "step")
0 */4 * * *     /path/to/command
```
## Crontab Format
User cron jobs are stored in a "crontab" file at: `/var/spool/cron/username`
- Each line of the crontab file has 6 fields.
- cron uses 24 hour time
- As usual, lines beginning with `#` are comments.

Crontab fields:

    Minute       0-59
    Hour         0-23
    Day of month 1-31
    Month        1-12 (or use names: Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec)
    Day of week  0-7  (0 or 7 is Sunday, or use names: Sun Mon Tue Wed Thu Fri Sat)
    Command to run
    
    An asterisk * in a field means "any"


Instead of the first five fields, one of eight special strings may appear:

    @reboot       Run once, at startup.
    @yearly       Run once a year, "0 0 1 1 *".
    @annually     (same as @yearly)
    @monthly      Run once a month, "0 0 1 * *".
    @weekly       Run once a week, "0 0 * * 0".
    @daily        Run once a day, "0 0 * * *".
    @midnight     (same as @daily)
    @hourly       Run once an hour, "0 * * * *".

Notes: 
- **system** crontabs have an extra field, indicating the user.
- Cron does **not** deal with **seconds**, you can't have cron jobs running every 30 seconds. 
- Watch out using "day of month" and "day of week" together. This becomes an "or" condition not an "and" condition.
- crontabs should end with a newline character. This will be assured if you use `crontab -e`
- If your command has any `%` symbols, it will fail because cron interprets `%` as newline. You need to quote or escape `%`
- Some crons don't support ranges i.e 4-6

## Output

**Any** output **(both stdout and stderr)** of a command or script that cron runs will
be sent to you by email, if your system has a "sendmail" command.  
To stop this, redirect output, as follows:

```bash
# Only send mail if there are errors
    0   *   *   *   *  /some/command   >/dev/null
# Suppress all output, never send mail
    0   *   *   *   *  /some/command   >/dev/null 2>&1
```
## Cron GOTCHAs
Sometimes a command or script will work from the terminal but fail in a cron job.  
Cron jobs execute in a **limited** environment, not the same as your login shell.

For example: this is the result of running `printenv` in a user cron job

```bash
HOME=/home/fred
LOGNAME=fred
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
LANG=en_US.UTF-8
SHELL=/bin/sh
PWD=/home/fred
```
Compare this with running `printenv` from the terminal.

Most often, the problem is with the `PATH`.  
To fix it, do one or more of the following

- Use absolute paths to commands that are not in cron's PATH
- Put the desired `PATH` in the shell script that gets run by cron
- set the `PATH` variable in the crontab file, this will apply to all cron jobs.

```bash
# Use an absolute path
/opt/app/bin/some_app

# Add to existing path in a shell script
PATH=/opt/app/bin:$PATH
some_app

# Set path in a crontab. you need to specify the entire PATH
PATH=/opt/app/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
0 1 * * * some_app

# You CANNOT do this in a crontab
PATH=/opt/app/bin:$PATH
```

If your script uses the `$USER` environment variable, it won't work in cron. This variable does not exist in the cron environment. Use `$LOGNAME` instead.

Beware of commands like **curl** and **rsync --progress** that give status output while the command is running. This **won't work** in cron. To fix.
- most commands have a "silent" or "quiet" option
- otherwise, redirect output to /dev/null

```bash
# curl example
curl --silent -o output_file http://example.com

# Redirect only stdout output, and not stderr
some-command >/dev/null

# Redirect both stdout and stderr
    some-command >/dev/null 2>&1
```

## Other Issues
- It's a pain to run a one-time cron job for testing.
- You have to edit the crontab file, test the cron job, then remember to edit the crontab again.
- using the `at` command is a workaround, but `at` is a completely different application, it's not cron, so you're not really testing cron

When cron sends email, there's no easy way to control the Subject line.
- It's going to be: `Cron <fred@bedrock> /path/to/script`
- And you just have to live with it.

Cron is not so good for machines that are not running all the time.
`anacron` is a method to run jobs on laptops and other machines that don't run all the time.

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

## How cron works

Cron is a deamon that is always running.
Cron searches its spool area (/var/spool/cron/crontabs) for crontab files
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
