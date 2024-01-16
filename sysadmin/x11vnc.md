# X11vnc
X11vnc shares the current desktop, even if the user is logged out.  
This is not true with other vnc servers such as Vino, TigerVNC

Install x11vnc server with package manager.

    sudo apt-get install x11vnc

Create a VNC password file.

    sudo x11vnc -storepasswd yourVNCpasswordHERE /etc/x11vnc.pass

Create file /etc/systemd/system/x11vnc.service

    sudo vi /etc/systemd/system/x11vnc.service

Put this in the file

    [Unit]
    Description=Start x11vnc at startup.
    After=multi-user.target

    [Service]
    Type=simple
    ExecStart=/usr/bin/x11vnc -xkb -auth guess -forever -loop -noxdamage -repeat -rfbauth /etc/x11vnc.pass -rfbport 5900 -shared

    [Install]
    WantedBy=multi-user.target

Enable the service:

    sudo systemctl enable x11vnc.service
    sudo systemctl daemon-reload

Reboot.

x11vnc now runs automatically at bootup

NOTE: Sometimes, when you connect, you get a black screen. Restart the
display manager. You need to ssh into the box.

    sudo systemctl restart lightdm.service

## Optional - don't run x11vnc at boot
- Disable systemd service `sudo systemctl disable x11vnc.service`
- Stop x11vnc `sudo systemctl stop x11vnc.service`

To start x11vnc for a one time connection, via ssh
```
# Check if x11vnc is already running
ssh user@host pgrep x11vnc && echo "x11vnc is running"

# stop x11vnc if running
ssh -t use@host sudo pkill x11vnc

# Start x11vnc if not running.
ssh -t user@host sudo /usr/bin/x11vnc -quiet -xkb -auth guess -noxdamage -display :0 -rfbauth /etc/x11vnc.pass -rfbport 5900
```
Connect with vnc client such as Remmina. x11vnc should exit when Remmina disconnects.

## x11vnc on Raspberry Pi OS
The default ssh server doesn't work with remmina. it's easier to just use x11vnc
- run `sudo raspi-config` and disable the vnc server.
- install x11vnc: `sudo apt install x11vnc`

### If pi is set to autologon to the Desktop
- run `x11vnc -usepw -forever`
- it asks for password, do what it says, save the password where it wants.
- the vnc server should be running, test it with remmina on another machine. 
- you can start x11vnc automatically at boot by adding a .desktop file to the user's autostart directory.

```bash
cd ~/.config
[ -d autostart ] || mkdir autostart
cd autostart
nano x11vnc.desktop
```
Put this:
```
[Desktop Entry]
Type = Application
Name = x11vnc
Exec = x11vnc -usepw -forever
StartupNotify = false
```
### If the pi is NOT set to autologon
configure the systemd service as above.

### If the pi runs headless
- the display size will be tiny.
- run `sudo raspi-config` ... under "Display Options" ... "VNC Resolution".

## x11vnc with novnc (untested)

/etc/systemd/system/x11vnc.service

    [Unit]
    Description=X11VNC Server
    After=multi-user.target network.target

    [Service]
    Restart=always
    ExecStart=/usr/bin/x11vnc -create -xkb -noxrecord -noxfixes -noxdamage -display :0 -auth /var/run/lightdm/root/:0 -nopw -rfbport 5900 -nevershared -forever -reopen

    [Install]
    WantedBy=multi-user.target

/etc/systemd/system/novnc.service

    [Unit]
    Description = start noVNC service
    After=syslog.target network.target x11vnc.service

    [Service]
    Type=simple
    User=root
    ExecStart=/snap/novnc/current/utils/launch.sh --vnc localhost:5900 --listen 6080

    [Install]
    WantedBy=multi-user.target

Enable your services:

    systemctl enable x11vnc; systemctl enable novnc; service start novnc; service start x11vnc

Set up proxying - this is from /etc/apache2/apache2.conf

    ProxyVia On ProxyPass /novnc http://localhost:6080 ProxyPassReverse /novnc http://localhost:6080 ProxyPass /websockify ws://localhost:6080 retry=3 ProxyPassReverse /websockify ws://localhost:6080
