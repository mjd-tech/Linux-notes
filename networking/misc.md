# Linux Network Administration

© 2016 [Martin Bruchanov](http://bruxy.regnet.cz/), bruchy@gmail.com

## IPv4 CIDR, subnet mask

CIDR  | Subnet mask      | # of IP addresses  | # of usable IP addresses
---   | ---              | ---                | ---
/32   | 255.255.255.255  | 1                  | 1
/31   | 255.255.255.254  | 2                  | 2*
/30   | 255.255.255.252  | 4                  | 2
/29   | 255.255.255.248  | 8                  | 6
/28   | 255.255.255.240  | 16                 | 14
/27   | 255.255.255.224  | 32                 | 30
/26   | 255.255.255.192  | 64                 | 62
/25   | 255.255.255.128  | 128                | 126
/24   | 255.255.255.0    | 256                | 254

## Reserved IP addresses

Description            | IPv4                                      | IPv6
---                    | ---                                       | ---
Loopback               | 127.0.0.1/8                               | ::1/128
Unspecified address    | 0.0.0.0/8                                 | ::/128
Multicast              | 224.0.0.0/4                               | ff00::/8
Private                | 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 | fc00::/7
Zeroconf automatic IP  | 169.254.0.0/16                            |

## Basic network setup

- Manage networking:
  - Systemd: `systemctl start/stop/restart NetworkManager`
- Set hostname:
  - `hostname *name*`
  - `nmcli general hostname *name*`
  - edit file `/etc/hostname`
  - `hostnamectl set-hostname *name*`
- Check hostname: `hostname`, `hostnamectl`
- Show devices and configuration: `ip addr `, `ip link`
- Disable device: `ip link set eth0 down`; `nmcli con down eth0`
- Rename device (when disabled): `ip link set *enp0s25* name *eth0*`
- Enable device: `ip link set eth0 up`; `nmcli con up eth0`
- Set IP address:
  - `ip addr add 192.168.1.2/24 dev eth0`
  - `dhclient -v eth0`
- Delete IP address: `ip addr del 192.168.1.2/24 dev eth0`
- Add alias interface: `ip addr add 10.0.0.1/8 dev eth0 label eth0:1`
- Set promiscuous mode: `ip link set eth0 promisc on/off`
- Change MAC address: `ip link set dev eth0 address AA:BB:CC:DD:EE:FF`
- Routing:
  - `ip route add default via 192.168.1.1 dev eth0`
  - `ip route add 192.168.1.0/24 via 192.168.1.1`

## Wi-Fi Networking

- Scan available networks: `iwlist wlan0 scan`; `nmcli dev wifi`
- Display available channels: "iwlist wlan0 freq"
- Connect with WEP network: "iwconfig wlan0 essid *"Network SSID"* key
  *HEX_KEY*"
- Connect with WEP network: "iwconfig wlan0 essid *"Network SSID"* key
  s:*ASCII_KEY*"
- Connect with WEP network: "nmcli dev wifi connect *"Network SSID"*
  password '123...'"
- Watch signal quality: "watch -n 1 cat /proc/net/wireless " (link =
  SNR, level in dBm)

## NetworkManager, nmcli, nmtui

- Text user interface for NetworkManager: "nmtui"
- Manage service: "systemctl enable/disable/start/restart/stop
  NetworkManager.service"
- List all devices: "nmcli dev status"
- List all connections: "nmcli connection show"
- Show detail about connection: "nmcli con show eth0"
- Add connection: "nmcli con add con-name "default" type ethernet ifname
  eth0"
- Set IPv4: "nmcli con add con-name "static" ifname eth0 autoconnect no
  type ethernet ip4 172.125.X.10/24 gw 172.25.X.254"
- Set IPv4: "nmcli connection modify eth0 ipv4.addresses 10.0.0.2/8
  ipv4.gateway 10.0.0.1"
- Activate/deactivate connection: "nmcli con up/down "static""
- Reload configuration: "nmcli con reload"
- Disconnect interface and disable autoconnect: "nmcli device disconnect
  *DEV*"
- Disable all managed interfaces: "nmcli net off"
- Add, modify, delete connection: "nmcli con add / mod "ID" / del "ID""
- Set DNS: "nmcli con modify eth0 ipv4.dns "8.8.8.8,8.8.4.4""
- Set routes: "nmcli connection modify eth0 ipv4.routes "192.168.0.0/24
  10.0.0.1, 192.168.1.0/24 10.0.0.1""

## DHCP (Dynamic Host Configuration Protocol)

- Configure device: "dhclient -v eth0"
- Release device configuration: "dhclient -r"
- DHCP client data: "/var/lib/dhclient/dhclient.leases"

## Network sockets of processes

- List active connections: "netstat -plunt"; "lsof -i"; "ss -tua"
- List all UNIX listening ports: "netstat -lx"
- Display all active network connection: "netstat -na"
- List process communication on port: "lsof -i :22" / "lsof -i :ssh"
- Check PID binded on local port: "ss -lt"; "fuser -n tcp 22"
- Monitor net. communication of single process: "strace -f -e
  trace=network -s 10000 -p *PID*"
- Color and interactive network monitor: "iptraf-ng"

## ICMP (Internet Control Message Protocol)

- For IPv6 use: "ping6", "tracepath6", "traceroute6"
- Ping n-times: "ping -c *n* IP"
- Broadcast: "ping -b 10.0.0.255"
- Use different interface: "ping -I *eth1*"
- Trace route: "traceroute *host*"; "mtr -c 1 -r *host*";
- Use TCP instead: "tcptraceroute", "tcping *host port*"

## Ethernet Bridge Manipulation

- Shows all current instances of the ethernet bridge: "brctl show"
- Create bridge "*br0*": "brctl addbr br0", "nmcli con add type bridge
  ifname br0"
- Add/remove interface: "brctl addif br0 eth1" / "brctl delif br0 eth1"
- Enable/disable Spanning Tree Protocol (STP): "brctl stp br0
  on" / "off"
- Delete bridge: "brctl delbr br0"

## ARP (Address Resolution Protocol)

- Show ARP table: "arp"; "ip neighbor list"; "cat /proc/net/arp"
- Clean ARP table: "ip -s neigh flush all"
- Add an entry in your ARP table:
  - "arp -i eth0 -s 192.168.0.1 00:11:22:33:44:55"
  - "ip neigh add 192.168.0.1 lladdr 00:11:22:33:44:55 nud permanent dev
    eth0"

## Routing

- Display routes: "ip route show", "ip route list", "netstat -rn"
- Set default gateway: "ip route add default via 192.168.1.1", "route
  add default gw 192.168.1.1"
- Print host interfaces and routes: "nmap --iflist"
- Route IP range through eth0: "ip route add 192.168.1.0/24 dev eth0"
- Delete route: "ip route delete 192.168.1.0/24 dev eth0"
- Enable IP forwarding:
  - "echo "1" \> /proc/sys/net/ipv4/ip_forward"
  - Save in "/etc/sysctl.conf" option "net.ipv4.ip_forward = 1"
- Static route configuration:
  "/etc/sysconfig/network-scripts/route-eth0":
  - "default via 10.254.0.1 dev eth0"
  - "172.31.0.0/16 via 10.254.0.1 dev eth0"

## Firewall

## "tcpdump" – dump traffic on a network

- Display communication with HTTP: "tcpdump -i eth0 'tcp port 80'"
- Communication with HTTP, print all ASCII, truncate packet content to
  1024 bytes: "tcpdump -vvv -s 1024 -l -A 'tcp port http'"
- Display all communication except SSH: "tcpdump -i eth0 'not port ssh'"
- Display frames at the data link layer: "tcpdump -e"
- Don't convert host addresses / ports to name: "tcpdump -n / -nn"
- Hexdump headers and data of each packet: "-X", and header "-XX"
- Monitor source: "tcpdump -i eth0 src 192.168.10.1"
- Monitor destination: "tcpdump -i eth0 dst 192.168.10.1"
- Monitor network: "tcpdump -i eth0 net 192.168.10.1/24"
- DNS packets: "tcpdump udp and src port 53"
- Capture communication on "eth1" to file: "tcpdump -ni eth1 -w
  file.cap"
- Capture telnet and ssh: "tcpdump -n portrange 22-23"
- Check packet filter syntax: "man pcap-filter"

## netcat – Concatenate and redirect sockets

- Connect to port 80: "nc www.google.com 80"
- netcat default port, if "-p" it is not specified: 31337
- Protocols: "--tcp", "--udp", "--sctp", "--ssl", "-4", "-6"
- Listen on TCP port 1234: "nc -v -k -l 1234", UDP port: "nc -v -k -ul
  1234"
- Allow/deny: "--allow 192.168.0.0/24", "--deny 10.0.0.0/8"
- Transfer file:
  - Sender: "cat file.txt \| nc -v -l -p 5555"
  - Receiver: "nc *host* 5555 \> file_copy.txt"
- Remote shell:
  - Server: "nc -v -l -e /bin/bash"
  - Client: "nc *host*", "telnet *host* 31337"
- Reverse telnet:
  - Computer with public IP: "nc -vv -l"
  - Computer behind firewall: "nc -v *host* -e /bin/bash"

## bash – network support for shell scripting

- Special filenames: "/dev/tcp*/host/port*", "/dev/udp*/host/port*"
- Open file descriptor 3 for TCP: "exec 3\<\>/dev/tcp/www.root.cz/80"
- Generate HTTP request: "echo -en "GET /unix/ HTTP/1.1\r\nHost:
  www.root.cz\r\n\r\n" \>&3"
- Read from file descriptor 3: "cat \<&3"
- Close descriptor 3: "exec 3\>&-"
- Check open descriptors for current shell: "ls -l /proc/\$\$/fd"

## Domain Name Service (DNS)

- Local names definition: "/etc/hosts"
- Sources of name resolution: "/etc/nsswitch.conf"
- Resolver configuration file – "/etc/resolv.conf":

    nameserver 8.8.8.8
    nameserver 8.8.4.4
    search .mydomain.com

- Look up the IP address: "host *name*", "nslookup *name*", "dig +short *name*"
- Get DNS record: "dig *name*", "host -a *name*"
- Get entries from Name Service Switch libraries: "getent"
- Test resolution with "/etc/hosts": "getent hosts *name*"
- Return hostname for IP: "dig -x 10.32.1.10 +short"
- Return IP for *hostname*: "dig *hostname* +short"
- Scan network for DNS records: "for i in 192.168.10.{1..254"; do echo
  -e //i \\t //(dig +short -x \$i); done
- Get specific DNS record: "dig -t *record* *hostname/domain*", "host -t
  *record* *hostname/domain*"
  - "A / AAAA" – return 32/128 bit address for host
  - "CNAME" – aliases of hostname, can point to A
  - "MX" – mail exchanger record
  - "NS" – specify authoritative nameserver for domain
  - "PTR" – pointer records for reverse lookup (addr-\>host)
  - "SOA" – Start of Authority, name of the server that supplied the
    data for the zone
- User given DNS server 8.8.8.8: "dig @8.8.8.8 *hostname*", "host
  *hostname* 8.8.8.8"
- Ask root name server for a record: "dig @a.root-servers.net
  example.com" (will return authority DNS for domain)

## WHOIS service

- Client to access WHOIS service: "whois", "jwhois"
- Query domain on given WHOIS server: "whois -h whois.nic.cz seznam.cz"
- Check who owns current IP address/domain: "whois *IP*" / "whois
  *domain*"

## HTTP(S) (Hypertext Transfer Protocol \[SECURE\])

- URL format:
  "[http://user:password@domain:port/path?query#fragment_id](http://user:password@domain:port/path?query#fragment_id)"
- Mirror site: "wget -e robots=off -r -L *URL*",
- Display HTTP header: "curl -I *URL*", "wget -S *URL*"
- Download file: "curl -O *URL/file*"
- Write output to file: "curl -o file *URL* "
- List directory: "curl -s *URL* --list-only"
- Save cookies to file: "wget -q --cookies=on --keep-session-cookies
  --save-cookies=cookie.txt *URL*"
- Use saved cookies: "wget -nv --content-disposition --referer=
  --cookies=on *URL*"
- Download URL and display it in stdout: "curl *URL*", "wget -q -O -
  *URL*"
- Resume broken download: {wget -c *URL*, "curl -L -O -C - *URL*"
- Change referer and browser id.: "wget --referer *URL* --user-agent
  "Mozilla/5.0 (compatible; Linux)""
- Set HTTP header: "curl -H "Content-Type: application/xml" *URL*"
- Send cookie: "curl -H "Cookie: name1=value; name2=another" *URL*",
  "curl --cookie "name1=value; name2=another" *URL*"
- POST request: "curl -X POST -d 'name1=value&name2=another' *URL*"
- Form upload file: "Z curl --form upload=@*localfilename* --form
  press=OK *URL*"
- Enable HTTP proxy in shell: "export
  http_proxy=<http://foo:bar@202.54.1.1:3128/>"
- Use the same for HTTPS: "export https_proxy=\$http_proxy"
- Convert page to text: "elinks -dump *URL*"

## OpenSSL

- Generate random sequences: "openssl rand -base64 8"
- Display server certificate: "openssl s_client -showcerts -connect
  google.com:443"
- Generate certificate: "openssl req -x509 -nodes -days 365 -newkey
  rsa:2048 -keyout server.key -out server.crt"
- Check SSL key MD5: "openssl rsa -noout -modulus -in server.key \|
  openssl md5"
- Check expiration date: "echo \| openssl s_client -showcerts -connect
  google.com:443 2\>&1 \| openssl x509 -noout -dates"

## Network Time Protocol (NTP)

- NTP query program: "ntpq tik.cesnet.cz"
- Get server variables: "ntpq -i tik.cesnet.cz \<\<\< "cl""
- Show network time synchronisation status: "ntpstat"
- Set date from server: "ntpdate -s time.nist.gov"


## E-mail

- Send email: "curl --mail-from blah@test.com --mail-rcpt foo@test.com
  smtp://mailserver.com"
- Send email: "mail -s "This is subject" foo@test.com"
