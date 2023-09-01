# Zero Config Networking
Zeroconf attempts to automatically create a usable IP network without manual intervention.

1.  automatic IP address assignment (without a DHCP server),
2.  distributed name resolution using mDNS (without a DNS server)
3.  network service publishing/discovery using DNS-SD.

Common Implementations are Bonjour (Apple), Avahi (Linux).  
MS Windows has some support, but not fully integrated

## IP Address Assignment
In the absense of a DHCP server, both IPv4 and IPv6 include Link-Local address
assignment (RFC 3927).

- IPv4 uses the special block 169.254.0.0/16
- IPv6 hosts use the prefix fe80:: combined with a 64 bit host address
  derived from the 48-bit MAC address.

The host is required to ensure that it generates a unique address.  
IPv4 hosts use link-local addressing only as a last resort when a DHCP server is unavailable.  
IPv6 hosts are required to configure a link-local address even when global addresses are available.

## Name resolution

Multicast Domain Name Service (mDNS) allows a device to announce itself
using a special multicast IP address. The domain is ".local" by default

When a client wants to know the IP address of a computer given its name,  
it sends a request to 224.0.0.251 (IPv4) or ff02::fb (IPv6)  
The computer with the corresponding name replies with its IP address.

## Service Discovery

DNS-SD allows clients to discover a named list of services by type in a
specified domain using standard DNS queries.

## Practical Applications
You can resolve any participating host by specifying `hostname.local`

"Service Discovery" is useful in theory, unfortunately not too many appplications support it.

Most Linux file managers, i.e. Nautilus, Nemo, Caja will discover SFTP and SMB services
and list them in "Browse network". The printer configurator will find cups-enabled printers.
Various other specialty apps will find their specialty services.

avahi-daemon does not support CNAME (host alias) records.
While there is a /etc/avahi/hosts file, it cannot be used to
similate CNAME records, it is used to map hosts that are not mDNS aware.

### Find all Zeroconf devices on a network

You cannot rely on tools like `avahi-browse`  
Some zeroconf devices (such as Ubuntu desktop) do not by default announce services,
but still partake in mDNS (hostname.local to IP address mapping).

Since there is no central "DNS server", you have to query the mDNS multicast address (224.0.0.251),
one query for each active IP on your LAN.

This technique uses `nmap` to find the "alive" IP addresses, then `dig` to
resolve the IP addresses to host names:

```bash
for ip in $(nmap -n -sP 192.168.1.0/24 | awk '/report/ {print $5}'); do 
    echo -n "$ip :"
    dig +short -x $ip @224.0.0.251 -p 5353
    # or: drill -Qx $ip @224.0.0.251 -p 5353
done
```

- `nmap` scans the network and lists the "live" IP addresses
- `awk` filters out the extraneous output from nmap
- `dig` queries the multicast address for each host IP, and reports a
  host name (if any)
- `drill` if you don't have dig

### Monitor Zeroconf advertisements

    avahi-browse --all --terminate
    avahi-browse -at

For more detail:

    avahi-browse --all --resolve --terminate
    avahi-browse -art

To look for just SSH services use the following command. Note the rather
cryptic type name that you must use. You may also find it useful to add
the --resolve option.

    avahi-browse _ssh._tcp
    avahi-browse --resolve _ssh._tcp

A complete name of services is here:
http://www.dns-sd.org/ServiceTypes.html

### GUI browser

The *avahi-discover* package has a simple GUI browser. This is the
equivalent of the command-line tool avahi-browse.

### SFTP
If you install openssh-server, it does not automatically create the avahi "service discovery"  
You can enable it with:

    sudo cp /usr/share/doc/avahi-daemon/examples/sftp-ssh.service /etc/avahi/services/

If that example is not available then the following should work

/etc/avahi/services/sftp-ssh.service
```
<?xml version="1.0" standalone='no'?><!--*-nxml-*-->
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">

<service-group>
  <name replace-wildcards="yes">SFTP on %h</name>
  <service>
    <type>_sftp-ssh._tcp</type>
    <port>22</port>
  </service>
</service-group>
```
Make sure the first line has no leading spaces or tabs.

