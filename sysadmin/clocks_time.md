# Clocks and Time
A computer has two clocks, hardware (RTC) and System.

Linux hardare clock is set to UTC (Greenwich Mean Time).  
Windows sets it to localtime.

```bash
# Read system clock, RTC, show timezone, show NTP status
timedatectl

# List timezones
timedatectl list-timezones

# Set timezone
sudo timedatectl set-timezone America/New_York
# creates /etc/localtime symlink that points to a zoneinfo file under /usr/share/zoneinfo/
# Tip: The time zone can also be selected interactively with tzselect.

# Set system clock
sudo timedatectl set-time "yyyy-MM-dd hh:mm:ss"

# Set hardware clock
sudo timedatectl set-local-rtc 0

# Read hardware clock
sudo hwclock show

# Set hardware clock from system clock
sudo hwclock --systohc

# Set system clock
sudo timedatectl set-time "yyyy-MM-dd hh:mm:ss"
```

## Linux Commands

```bash
# Read System Clock:
date

# Set System Clock:
sudo date --set "2 OCT 2012 18:00:00"

# Note: the date command has numerous formatting options. see:
man date

# Read Hardware Clock:
sudo hwclock --show

# Set Hardware Clock:
sudo hwclock --set --date="2012-12-15 20:49:00"

# date must be in local time, even if Hardware Clock is UTC
Coordinated Universal time.

# Set the System Time from the Hardware Clock:
sudo hwclock --hctosys

# Set the Hardware Clock to the current System Time:
sudo hwclock --systohc  
```

### Timezone

In Debian/Ubuntu systems

    sudo dpkg-reconfigure tzdata

This process will set up `etc/localtime` using the appropriate file from
`/usr/share/zoneinfo`

### ntpdate

This command syncs the system clock to a ntp (network time protocol)
server.  
In Ubuntu / Linux Mint, sync occurs during network interface init
(bootup or coming back from suspend).  
See `/etc/network/if-up.d/ntpdate`

Default ntp server is ntp.ubuntu.com  
See `/etc/default/ntpdate`

### ntpd

Daemon process, continually syncs system clock. Not installed by default
in Ubuntu / Linux Mint desktops. A good idea to use for servers which
stay up all the time.

## Windows and UTC

To dual boot Windows and Linux, it's best to configure Windows to use UTC.

Using regedit, add a DWORD value with hexadecimal value 1 to the registry:

    HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation\RealTimeIsUniversal

Alternatively, create a .reg file (on the desktop) with the following
content and double-click it to import it into registry:

    Windows Registry Editor Version 5.00

    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation]
        "RealTimeIsUniversal"=dword:00000001


## Resources

- [The Clock Mini-HOWTO: How Linux Keeps Track of
  Time](http://tldp.org/HOWTO/Clock.html#toc2)
- [System Time](https://en.wikipedia.org/wiki/System_time)
