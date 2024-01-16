# Networking
Linux desktop systems mostly use NetworkManager. Some systems use connman.

```bash
# ping and nmap
ping -c4 google.com         # quit after 4 trys. either IP6 or IP4
ping -4 -c4 google.com      # force IP4
ping -6 -c4 google.com      # force IP6
tracepath                   # traces path to a network host. was traceroute
nmap -sn 192.168.1.0/24     # scan for hosts
sudo nmap -sn 192.168.1.0/24    # gets MAC addr and NIC mfg.
sudo nmap -O 192.168.1.1    # attempt to detect O/S
nmap -p 80 192.168.1.1      # ping a tcp port
nmap -p U:53 192.168.1.1    # udp port

ip -br addr     # ip address, brief
ip -br link     # mac address
ip -br neigh    # ip and mac of other devices on LAN
ip route

getent hosts myhost     # includes /etc/hosts in the lookup
getent ahostsv4 myhost  # IP4 only, ahostsv6 for IP6 only
drill                   # similar to dig, ex. drill -Q MX google.com
whois example.com       # domain registration info
# these are deprecated.
host                    # get ip address of hostname. (install "bind" package)
dig                     # dns lookup, ex. dig +short MX google.com ("bind" package)

avahi-browse -at                # scan LAN for hosts with mDNS announcements
avahi-resolve -4n myhost.local  # get IP4 address via mDNS. -6 for IP6
avahi-resolve -4a 192.168.1.1   # get hostname via IP4 mDNS. -6 for IP6

# NetworkManager
systemctl start/stop/restart/status NetworkManager
nmcli con   # add "show" for more details
nmcli dev   # add "show" for more details
nmtui   # for editing connections, use nmcli for status.
```

## Network Interface Names

Linux since 2012 uses "Consistent Network Device Naming".  
Example: enp2s0 means: "**e**ther**n**et, **P**CI bus **2**, **s**lot **0**".

To get the old-style "eth0" and "wlan0" back:

```bash
sudo nano /etc/default/grub
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"

# Generate new grub.cfg file
sudo grub-mkconfig -o /boot/grub/grub.cfg

# NOTE: biosdevname is specific to Dell hardware, so it's probably not needed.
# but including it doesn't seem to be a problem on non-Dell hardware
```
## Additional IP address(es) on an interface
```bash
sudo ip address add 192.168.1.2/24 dev eth0
```

## Debian-Based systems
- NetworkManager with ifupdown plugin
- /etc/netplan/ directory, with a file that says to use NetworkManager.
- /etc/network/ directory, with several "up" and "down" scripts.
- DNS: systemd-resolved
- /etc/resolv.conf: nameserver 127.0.0.53
- actual nameservers in use: `resolvectl status`
- Raspberry Pi OS does not use netplan or systemd-resolved

## Arch-Based systems
- NetworkManager with no plugins
- no /etc/netplan or /etc/network directories
- DNS: NetworkManager
- /etc/resolv.conf: actual nameservers in use

## Copying NetworkManager settings
each connection configured in NetworkManager is stored in a file in
`/etc/NetworkManager/system-connections`
These files must be owned by root:root and chmod 600.

Openvpn connections may have a reference to external files. ex. `cert=/home/user/somedir/somefile.crt`

NetworkManager assigns a UUID to each connection based on the MAC address of the interface.
If you change hardware, NetworkManager won't use old connections because the UUID no longer
matches what it expects.

THIS IS UNTESTED:  
According to https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/configuring_and_managing_networking/assembly_manually-creating-networkmanager-profiles-in-keyfile-format_configuring-and-managing-networking
It appears the strategy is:

- copy the files, make sure root:root 600
- delete the UUID line from each file
- get the mac address of the network interfaces `ip -br link`
- add mac address to file
```
[ethernet]
mac-address=00:53:00:8f:fa:66

# or
[wifi]
mac-address=00:53:00:8f:fa:66
```
- reload connections: `nmcli connection reload`

It's not clear how or if this works for vpn connections, as there'e no mac address to add.

