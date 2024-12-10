# Laptops and other occasionally used devices
1. Schedule the backup job to run daily , same as the "always on" devices
2. Also, schedule the backup job to run at boot, but:
    - only once a day, even if rebooted multiple times.
    - wait for networking and other important stuff to start first

This is done with a combination of cron and anacron.

## set up anacron
```

mkdir -p ~/.anacron/{etc,spool}
cd ~/.anacron/etc
xed anacrontab

```
Replace "fred" with your username (3 places), and paste

```

HOME=/home/fred
LOGNAME=fred
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
LANG=en_US.UTF-8
SHELL=/bin/sh
PWD=/home/fred

# period  delay  job-identifier  command
1         5      backup          $HOME/restic/jobs/backup

```
edit your crontab with `crontab -e`:
```

# m  h  dom mon dow   command
  15 3  *   *   *     $HOME/restic/jobs/backup
  @reboot anacron -s -t $HOME/.anacron/etc/anacrontab -S $HOME/.anacron/spool

```
When the computer boots, cron runs anacron.
anacron checks if it has already run the backup job today.
If not, it waits 5 minutes, then runs the backup job.

If the computer is left on, cron will run the backup job daily at 3:15AM

## anacron options used:
-s  Serialize execution of jobs. Anacron will not start a new job before the previous one finished.
-t  Use specified anacrontab. Required to run anacron as non-root user.
-S  Use the specified spool directory to store timestamps in. Required to run anacron as non-root user.