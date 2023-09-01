# Cron
- edit crontab: crontab -e
- list crontab: crontab -l

Run daily at 3AM
```
# m   h   dom   mon   dow   command
  0   3   *     *     *     /home/fred/restic/backup
```
## cron examples
```
# Daily at 3:30AM
30 3 * * *      /path/to/command

# First day of the month at 7:00PM (note: cron uses 24 hour time)
0 19 1 * *      /path/to/command

# Tuesdays and Fridays at 4:30AM (using a "list")
30 4 * * Tue,Fri    /path/to/command

# Monday thru Friday at 11:45PM (using a "range")
45 23 * * Mon-Fri   /path/to/command

# Every 4 hours, on the hour (using a "step")
0 */4 * * *     /path/to/command
```
